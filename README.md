[По русски](https://github.com/imposibrus/openvpn/blob/master/README-ru.md)

# Installation

```bash
# clone this repo and enter the dir:
git clone https://github.com/imposibrus/openvpn.git
cd openvpn
# build OpenVPN server config:
docker-compose run --rm openvpn-udp ovpn_genconfig -u udp://DOMAIN_OR_IP_HERE -s 10.1.1.0/24 -N # see options explanation below
# generate server certificates (enter password, when asked. answer yes, when asked y/n):
docker-compose run --rm openvpn-udp ovpn_initpki
# start server (this command enable only UDP protocol):
docker-compose up -d openvpn-udp
# if you whant to use TCP protol, start another container:
docker-compose up -d openvpn-tcp
# both container can be used simultaneously.

# generate config for each client:
export CLIENTNAME="your_client_name" # use only latin letters, numbers, - and _
# enter same password, as from previous command:
docker-compose run --rm openvpn-udp easyrsa build-client-full $CLIENTNAME
docker-compose run --rm openvpn-udp ovpn_getclient $CLIENTNAME > $CLIENTNAME.ovpn
# download $CLIENTNAME.ovpn to your device
```

## `ovpn_genconfig` options explanation

- `-u udp://DOMAIN_OR_IP_HERE` - OpenVPN server address. Only `udp://` can be used here;
- `-s 10.1.1.0/24` - specify clients subnet;
- `-N` - automatically add NAT-rules with iptables.

## For Mikotik clients

### Few config incompatibilities

You should comment `tls-auth` and `comp-lzo` in config manually:
- Open `conf/openvpn.conf`, find and comment (insert # before) this lines:
    ```conf
    # tls-auth /etc/openvpn/pki/ta.key
    # comp-lzo no
    ```
- Restart containers:
    ```
    docker-compose restart
    ```

### Configurig RouterOS client

1. Copy contents of `<key></key>`, `<cert></cert>` and `<ca></ca>` from your `$CLIENTNAME.ovpn` to separate files. You can use `.key`, `.cert` and `.ca.cert` extensions for this files;
2. Upload this 3 files to router via Files->Upload menu in WebFig or Winbox;
3. Go to System->Certificates menu and click Import button;
4. In File name menu select uploaded `.ca.cert` file, use `ovpn-server-ca` for Name, enter same Passphrase as you entered when generating config;
5. Repeat previous step for `.cert` and `.key` files. Use `ovpn-server-cert` and `ovpn-server-key` as Name, use same Passphrase;
6. In System->Certificates menu you can see two new certifactes;
7. Go to Interfaces, click `+`, select `OVPN Client`;
8. In General tab enter interface name, in Dial Out configure connection:
    - Connect to: `your server IP or domain, as specified on generating config`;
    - Port: `1194`;
    - Protocol: `use protocol for your RouterOS. version 6 - TCP, version 7 - UDP`;
    - User: `ovpn-user`;
    - Certificate: `ovpn-server-cert`;
    - Use peer DNS: `no`
9. Save and check connection (logs usefull for debugging).

### RouterOS 6

RouterOS 6 can connect only by TCP protocol, so start proper container, before connecting.

### Improved security (optional)

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

