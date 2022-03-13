```bash
docker-compose run --rm openvpn-udp ovpn_genconfig -u udp://DOMAIN_OR_IP_HERE -s 10.1.1.0/24 -N
docker-compose run --rm openvpn-udp ovpn_initpki
docker-compose up -d openvpn-udp
export CLIENTNAME="your_client_name"
docker-compose run --rm openvpn-udp easyrsa build-client-full $CLIENTNAME
docker-compose run --rm openvpn-udp ovpn_getclient $CLIENTNAME > $CLIENTNAME.ovpn
```

See more usage examples [here](https://github.com/kylemanna/docker-openvpn/blob/master/docs/docker-compose.md).


