# Настройки Openwrt для переключения в работу моста.

![Static Badge](https://img.shields.io/badge/build-passing-brightgreen)


IP адреса будут передаваться от DCHP-сервера "сверху"

Полезно для CPE220, например.

Сначала настраиваем Wi-Fi через Web-Gui (luci), потом подключаемся по SSH.
Далее vim, nano и т.д.

## Конфигурация /etc/config/network


``` bash
config interface 'loopback'
        option device 'lo'
        option proto 'static'
        option ipaddr '127.0.0.1'
        option netmask '255.0.0.0'

config globals 'globals'
        option ula_prefix 'fd00:ab:cd::/48'

config device
        option name 'br-lan'
        option type 'bridge'
        list ports 'eth0'
        list ports 'eth1'

config interface 'lan'
        option device 'br-lan'
        option proto 'none'
        option ipv6 '0'

config interface 'wan'
        option device 'br-lan'
        option proto 'dhcp'
        option peerdns '1'
        option delegate '0'

config interface 'wan6'
        option device 'br-lan'
        option proto 'dhcpv6'
        option reqaddress 'try'
        option reqprefix 'auto'
        option delegate '0'
```

## Отключить DHCP-сервер в /etc/config/dhcp

```bash

config dhcp 'lan'
        option interface 'lan'
        option ignore '1'

```

## Настроить фаервол в /etc/config/firewall

``` bash

config zone
        option name 'lan'
        option network 'lan'
        option input 'ACCEPT'
        option output 'ACCEPT'
        option forward 'ACCEPT'

```
## Перезагрузка сети

```bash
/etc/init.d/network restart
/etc/init.d/firewall restart
```
или

```bash
reboot
```

## конфигурация для /etc/config/wireless

Но! лучше настроить пердварительно через WEB
```bash
config wifi-device 'radio0'
        option type 'mac80211'
        option channel 'auto'
        option hwmode '11a'
        option path 'pci0000:00/0000:00:00.0'
        option htmode 'VHT80'
        option country 'RU'  # Укажите вашу страну
        option txpower '20'  # Мощность передачи (dBm)
        option disabled '0'  # Включить радио

config wifi-iface 'default_radio0'
        option device 'radio0'
        option network 'lan'  # Привязка к мостовому интерфейсу
        option mode 'ap'      # Режим точки доступа
        option ssid 'CPE220-OpenWRT'  # Имя сети
        option encryption 'psk2'       # WPA2-PSK
        option key 'your_strong_password'  # Пароль
        option isolate '0'    # Отключить изоляцию клиентов
        option wmm '1'        # Включить WMM (QoS)
        option ieee80211r '0' # Fast Transition (отключено)
```
