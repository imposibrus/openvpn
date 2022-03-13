# Installation

```bash
# build OpenVPN server config:
docker-compose run --rm openvpn-udp ovpn_genconfig -u udp://DOMAIN_OR_IP_HERE -s 10.1.1.0/24 -N # see options explanation below
# generate server certificates:
docker-compose run --rm openvpn-udp ovpn_initpki
# start server:
docker-compose up -d openvpn-udp

# generate config for each client:
export CLIENTNAME="your_client_name"
docker-compose run --rm openvpn-udp easyrsa build-client-full $CLIENTNAME
docker-compose run --rm openvpn-udp ovpn_getclient $CLIENTNAME > $CLIENTNAME.ovpn
# download $CLIENTNAME.ovpn to your device
```

## `ovpn_genconfig` options explanation

- `-u udp://DOMAIN_OR_IP_HERE` - OpenVPN server address. Only `udp://` can be used here;
- `-s 10.1.1.0/24` - specify clients subnet;
- `-N` - automatically add NAT-rules with iptables.

## For Mikotik clients

If you want to increase your communication privacy, use `aes-256` cipher on router's side and specify this cipher in `docker-compose.yml`:
```yaml
# ...
command: "ovpn_run --cipher AES-256-CBC --keysize 256"
# ...
command: "ovpn_run --cipher AES-256-CBC --keysize 256 --proto tcp"
# ...
```

## N.B.

Do not forget to enable the IPv4 forwarding. [How to](https://linuxconfig.org/how-to-turn-on-off-ip-forwarding-in-linux).



See more usage examples [here](https://github.com/kylemanna/docker-openvpn/blob/master/docs/docker-compose.md).

