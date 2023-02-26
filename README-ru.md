[English version](https://github.com/imposibrus/openvpn/blob/master/README.md)

# Установка

```bash
# склонируйте репозиторий и перейдите в созданную папку:
git clone https://github.com/imposibrus/openvpn.git
cd openvpn
# генерируем конфиг OpenVPN сервер:
docker-compose run --rm openvpn-udp ovpn_genconfig -u udp://DOMAIN_OR_IP_HERE -s 10.1.1.0/24 -N # расшифровка опций ниже
# генерируем сертификат сервера (введите пароль, если спросят. отвечайте yes, если спросят yes/no):
docker-compose run --rm openvpn-udp ovpn_initpki
# запускаем сервер (это команда запускает сервер ТОЛЬКО для протокола UDP):
docker-compose up -d openvpn-udp
# если вы хотите использовать протокол TCP, запустите другой контейнер:
docker-compose up -d openvpn-tcp
# оба контейнера могут работать одновременно.

# генерируем конфиг для каждого клиента:
export CLIENTNAME="your_client_name" # используйте ТОЛЬКО латинские буквы, цифры, - и _
# когда спросят, введите тот же пароль, что и вводили в предыдущих командах:
docker-compose run --rm openvpn-udp easyrsa build-client-full $CLIENTNAME
docker-compose run --rm openvpn-udp ovpn_getclient $CLIENTNAME > $CLIENTNAME.ovpn
# скачайте $CLIENTNAME.ovpn на устройство
```

## Расшифровка опций `ovpn_genconfig`

- `-u udp://DOMAIN_OR_IP_HERE` - Адрес сервера OpenVPN. Обязательно должен использоваться префикс `udp://`;
- `-s 10.1.1.0/24` - подсеть для клиентов в которой каждый будет получать IP-адресс;
- `-N` - автоматически добавить NAT-правила с помощью iptables.

## Для пользователей Mikrotik

### Правки в конфиге сервера

Вы должны закомментировать строчки с `tls-auth` и `comp-lzo` в файле конфигурации:
- Откройте в редакторе файл `conf/openvpn.conf`, найдите и закомментируйте (добавьте символ # вначале строки) следующие строчки:
    ```conf
    # tls-auth /etc/openvpn/pki/ta.key
    # comp-lzo no
    ```
- Перезапустите контейнеры сервера OpenVPN:
    ```
    docker-compose restart
    ```

### Настройка клиента для RouterOS

1. Скопируйте содержимое блоков `<key></key>`, `<cert></cert>` и `<ca></ca>` из файла `$CLIENTNAME.ovpn` в отдельные файлы. Для этих файлов можете использовать расширения `.key`, `.cert` и `.ca.cert`;
2. Загрузите эти три файла на роутер через меню Files->Upload в WebFig или Winbox;
3. В меню System->Certificates нажмите кнопку Import;
4. В поле File name выберите загруженный файл `.ca.cert`, в поле Name введите `ovpn-server-ca`, в поле Passphrase введите тот же пароль, что вводили для генерации конфига на сервере;
5. Повторите предыдущий шаг для файлов `.cert` и `.key`. В поле Name введите `ovpn-server-cert` и `ovpn-server-key` соответственно, используйте то же значение для поля Passphrase;
6. В списке System->Certificates должно было появится 2 новых сертификата;
7. В меню Interfaces, нажмите `+`, выберите `OVPN Client`;
8. Во вкладке General укажите название для интерфейса, во вкладке Dial Out настройте подключение:
    - Connect to: `IP или домен вашего сервера, тот же, что использовался при генерации конфига`;
    - Port: `1194`;
    - Protocol: `используйте поддерживаемый протокол в зависимости от версии RouterOS. для версии 6 - TCP, для версии 7 - UDP`;
    - User: `ovpn-user`;
    - Certificate: `ovpn-server-cert`;
    - Use peer DNS: `no`
9. Сохраните и проверьте подключение (в логах будут сообщения об ошибках подключения).

### RouterOS 6

RouterOS 6 может подключаться к серверу OpenVPN только по протоколу TCP, поэтому не забудьте запустить соответствующий контейнер перед подключением клиента.

### Опциональные улучшения в безопасности

Если хотите увеличить безопасность своего соединения, используйте шифрование `aes-256` в конфигурации OVPN client на роутере и укажите то же шифрование в файле `docker-compose.yml`:
```yaml
# ...
openvpn-udp:
    # ...
    command: "ovpn_run --cipher AES-256-CBC --keysize 256"
# ...
openvpn-tcp:
    # ...
    command: "ovpn_run --cipher AES-256-CBC --keysize 256 --proto tcp"
# ...
```

## P.S.

Не забудьте проверить ip forwarding на сервере. [Как включить](https://linuxconfig.org/how-to-turn-on-off-ip-forwarding-in-linux).



Больше примеров использования [тут](https://github.com/kylemanna/docker-openvpn/blob/master/docs/docker-compose.md).

