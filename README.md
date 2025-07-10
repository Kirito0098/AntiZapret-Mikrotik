# MikroTik WireGuard клиент — базовая настройка

## Пример обезличенного конфига

Ниже приведён пример обезличенного WireGuard-конфига для клиента.  
Замените значения на свои реальные ключи и адреса.

```ini
[Interface]
PrivateKey = "ПРИВАТНЫЙ_КЛЮЧ_СЕРВЕРА"
Address = 172.29.8.7/32
DNS = 172.29.8.1
Jc = 100
Jmin = 20
Jmax = 100
S1 = 0
S2 = 0
H1 = 1
H2 = 2
H3 = 3
H4 = 4

[Peer]
PublicKey = "ПУБЛИЧНЫЙ_КЛЮЧ_СЕРВЕРА"
PresharedKey = "PRESHARED_КЛЮЧ_СЕРВЕРА"
Endpoint = vpn.example.com:52443
AllowedIPs = 172.29.8.0/24, 172.30.0.0/15, 103.21.244.0/22, 103.22.200.0/22, 103.31.4.0/22, 104.16.0.0/12, 104.16.0.0/13, 104.24.0.0/14, 108.162.192.0/18, 131.0.72.0/22, 141.101.64.0/18, 162.158.0.0/15, 172.64.0.0/13, 173.245.48.0/20, 188.114.96.0/20, 190.93.240.0/20, 197.234.240.0/22, 198.41.128.0/17, 3.74.0.0/15, 34.0.0.0/16, 34.1.224.0/19, 35.207.0.0/16, 35.212.0.0/14, 35.217.0.0/18, 35.219.224.0/19, 54.193.0.0/16, 66.22.192.0/18
PersistentKeepalive = 15
```

---

## 1. Создать WireGuard интерфейс

Создаёт новый интерфейс WireGuard с указанным именем, MTU и приватным ключом.

```shell
/interface wireguard add name=wg-vpn mtu=1420 private-key="ВАШ_ПРИВАТНЫЙ_КЛЮЧ"
```

---

## 2. Назначить IP-адрес (использовать /24)

Присваивает IP-адрес интерфейсу WireGuard для локальной стороны VPN.

```shell
/ip address add address=172.29.8.7/24 interface=wg-vpn
```

---

## 3. Добавить пир (сервер) с полным списком allowed-address

Добавляет пир WireGuard (сервер), указывая публичный ключ сервера, опциональный pre-shared key, адрес и порт сервера, список разрешённых адресов и keepalive.

```shell
/interface wireguard peers add interface=wg-vpn \
    public-key="ПУБЛИЧНЫЙ_КЛЮЧ_СЕРВЕРА" \
    preshared-key="PRESHARED_KEY_ЕСЛИ_НУЖЕН" \
    endpoint-address=vpn.example.com \
    endpoint-port=52443 \
    allowed-address=\
        172.29.8.0/24,\
        172.30.0.0/15,\
        103.21.244.0/22,\
        103.22.200.0/22,\
        103.31.4.0/22,\
        104.16.0.0/12,\
        104.16.0.0/13,\
        104.24.0.0/14,\
        108.162.192.0/18,\
        131.0.72.0/22,\
        141.101.64.0/18,\
        162.158.0.0/15,\
        172.64.0.0/13,\
        173.245.48.0/20,\
        188.114.96.0/20,\
        190.93.240.0/20,\
        197.234.240.0/22,\
        198.41.128.0/17,\
        3.74.0.0/15,\
        34.0.0.0/16,\
        34.1.224.0/19,\
        35.207.0.0/16,\
        35.212.0.0/14,\
        35.217.0.0/18,\
        35.219.224.0/19,\
        54.193.0.0/16,\
        66.22.192.0/18 \
    persistent-keepalive=15s
```

---

## 4. Добавить NAT-маскарадинг для локальной сети

Включает маскарадинг (NAT) для исходящего трафика из локальной сети через VPN-интерфейс.

```shell
/ip firewall nat add chain=srcnat action=masquerade src-address=192.168.88.0/24 out-interface=wg-vpn
```

---

## 5. Добавить маршруты для всех подсетей через VPN

Добавляет статические маршруты для указанных подсетей через VPN-шлюз, с проверкой доступности шлюза.

```shell
/ip route add dst-address=172.29.8.0/24 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=172.30.0.0/15 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=103.21.244.0/22 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=103.22.200.0/22 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=103.31.4.0/22 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=104.16.0.0/12 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=104.16.0.0/13 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=104.24.0.0/14 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=108.162.192.0/18 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=131.0.72.0/22 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=141.101.64.0/18 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=162.158.0.0/15 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=172.64.0.0/13 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=173.245.48.0/20 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=188.114.96.0/20 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=190.93.240.0/20 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=197.234.240.0/22 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=198.41.128.0/17 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=3.74.0.0/15 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=34.0.0.0/16 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=34.1.224.0/19 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=35.207.0.0/16 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=35.212.0.0/14 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=35.217.0.0/18 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=35.219.224.0/19 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=54.193.0.0/16 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=66.22.192.0/18 gateway=172.29.8.1 distance=1 check-gateway=ping
```

---

## 6. Настроить DNS

Устанавливает DNS-сервер для клиентов, отключает получение DNS от провайдера и назначает VPN-DNS для DHCP.

```shell
/ip dns set servers=172.29.8.1 allow-remote-requests=yes
/ip dhcp-client set [find interface=ether1] use-peer-dns=no
/ip dhcp-server network set [find where address~"192.168.88.0/24"] dns-server=172.29.8.1
```

---

## 7. Отключить FastTrack

Отключает FastTrack для корректной работы VPN-трафика.

```shell
/ip firewall filter disable [find action=fasttrack-connection]
```

---

## Добавление новых подсетей

1. **Обновить список allowed-address пира (добавить новые сети через запятую):**

Добавляет новые подсети в список разрешённых адресов WireGuard-пира.

```shell
/interface wireguard peers set [find interface=wg-vpn public-key="ПУБЛИЧНЫЙ_КЛЮЧ_СЕРВЕРА"] allowed-address=\
    172.29.8.0/24,\
    172.30.0.0/15,\
    103.21.244.0/22,\
    ...,\
    НОВАЯ_ПОДСЕТЬ_1,\
    НОВАЯ_ПОДСЕТЬ_2
```

2. **Добавить маршруты для новых подсетей:**

Добавляет маршруты для новых подсетей через VPN-шлюз.

```shell
/ip route add dst-address=НОВАЯ_ПОДСЕТЬ_1 gateway=172.29.8.1 distance=1 check-gateway=ping
/ip route add dst-address=НОВАЯ_ПОДСЕТЬ_2 gateway=172.29.8.1 distance=1 check-gateway=ping
```

---

## Проверка работы

- **Пинг VPN-шлюза:**

  Проверяет доступность VPN-шлюза через WireGuard-интерфейс.

  ```shell
  /ping 172.29.8.1 interface=wg-vpn
  ```

- **Трассировка маршрута:**

  Проверяет маршрут до внешнего адреса через VPN.

  ```shell
  /tool traceroute 104.16.0.1
  ```

- **Просмотр трафика на VPN-интерфейсе:**

  Быстрый просмотр трафика на VPN-интерфейсе по порту 443.

  ```shell
  /tool sniffer quick interface=wg-vpn port=443
  ```

---

> **Если при первом подключении сайты по VPN открываются медленно или не открываются вовсе, попробуйте уменьшить MTU интерфейса WireGuard до 1380 (мне помогло):**
>
> ```shell
> /interface wireguard set [find name=wg-vpn] mtu=1380
> ```

---
