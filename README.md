# 🚀 Настройка WireGuard VPN на MikroTik: WinBox и Терминал

---

> ⚠️ **WireGuard поддерживается только в MikroTik RouterOS версии 7 и выше!**  
> Все действия и команды проверены на MikroTik RouterOS 7.19.3.
>  
> **ВНИМАНИЕ! Используйте свои ключи, адреса, параметры и свои сети из вашей конфигурации. Не копируйте значения из примера — замените их на свои!**

---

## 📑 Оглавление
- [🚀 Настройка WireGuard VPN на MikroTik: WinBox и Терминал](#-настройка-wireguard-vpn-на-mikrotik-winbox-и-терминал)
  - [📑 Оглавление](#-оглавление)
  - [📝 Что делает эта инструкция?](#-что-делает-эта-инструкция)
  - [💡 Пример конфигурации клиента WireGuard](#-пример-конфигурации-клиента-wireguard)
  - [🛠️ Подготовка к настройке](#️-подготовка-к-настройке)
  - [🧩 Пошаговая настройка](#-пошаговая-настройка)
    - [1️⃣ Создание интерфейса WireGuard](#1️⃣-создание-интерфейса-wireguard)
    - [2️⃣ Назначение IP-адреса интерфейсу](#2️⃣-назначение-ip-адреса-интерфейсу)
    - [3️⃣ Настройка подключения к серверу (peer)](#3️⃣-настройка-подключения-к-серверу-peer)
    - [4️⃣ Настройка NAT (маскарадинг)](#4️⃣-настройка-nat-маскарадинг)
    - [5️⃣ Перенаправление DNS-запросов](#5️⃣-перенаправление-dns-запросов)
    - [6️⃣ Скрипт автоматизации DNS, NAT И Добавление маршрутов](#6️⃣-скрипт-автоматизации-dns-nat-и-добавление-маршрутов)
    - [7️⃣ Настройка DHCP-клиента](#7️⃣-настройка-dhcp-клиента)
    - [8️⃣ Отключение FastTrack](#8️⃣-отключение-fasttrack)
    - [9️⃣ Важно про MTU и TCPMSS](#9️⃣-важно-про-mtu-и-tcpmss)
      - [🐧 На сервере WireGuard (Linux):](#-на-сервере-wireguard-linux)
      - [🛡️ На MikroTik:](#️-на-mikrotik)
  - [➕ Добавление новых подсетей](#-добавление-новых-подсетей)
  - [✅ Проверка работы](#-проверка-работы)
  - [❓ FAQ](#-faq)
  - [🛠️ Troubleshooting](#️-troubleshooting)
    - [1. Проблемы с MTU](#1-проблемы-с-mtu)
    - [2. Проблемы с DNS](#2-проблемы-с-dns)
    - [3. Проблемы с маршрутизацией](#3-проблемы-с-маршрутизацией)
    - [4. Проблемы с ключами](#4-проблемы-с-ключами)
    - [5. FastTrack не отключён](#5-fasttrack-не-отключён)
    - [6. Нет связи с сервером WireGuard](#6-нет-связи-с-сервером-wireguard)
  - [📚 Полезные ссылки](#-полезные-ссылки)

---

## 📝 Что делает эта инструкция?
- 🛡️ Создаёт интерфейс WireGuard для VPN
- 🏷️ Назначает IP-адрес интерфейсу
- 🔗 Настраивает peer (подключение к серверу)
- 🗺️ Настраивает маршруты и NAT для VPN-трафика
- 🌐 Перенаправляет DNS-запросы через VPN
- 🤖 Автоматизирует проверку состояния VPN и управление DNS
- 🚫 Отключает FastTrack для стабильной работы

---

## 💡 Пример конфигурации клиента WireGuard

> **Используйте свои ключи, адреса, параметры и свои сети из вашей конфигурации!**

Ниже пример конфигурационного файла WireGuard. Замените значения на свои (приватный ключ, публичный ключ, адрес сервера и т.д.).

```ini
[Interface]
PrivateKey = ПРИВАТНЫЙ_КЛЮЧ_СЕРВЕРА
Address = 172.29.8.7/32
DNS = 172.29.8.1

[Peer]
PublicKey = ПУБЛИЧНЫЙ_КЛЮЧ_СЕРВЕРА
PresharedKey = PRESHARED_КЛЮЧ_СЕРВЕРА
Endpoint = vpn.example.com:51443
AllowedIPs = 172.29.8.0/24, и так далее. 
PersistentKeepalive = 15
```

---

## 🛠️ Подготовка к настройке

> **ВНИМАНИЕ! Все ключи, адреса, параметры и сети должны быть взяты из вашей конфигурации. Не используйте значения из примера!**

1. **Для WinBox** 🖥️:
   - Скачайте и установите WinBox с сайта MikroTik.
   - Подключитесь к роутеру, введя IP-адрес, логин и пароль.

![Подключение к роутеру через WinBox](https://github.com/user-attachments/assets/ad9a128f-c9d8-43e9-b03c-b8bf076c6df3)
2. **Для терминала** 💻:
   - Подключитесь к роутеру через SSH или откройте терминал в WinBox (кнопка **New Terminal**).
3. **Проверьте ключи** 🔑: Убедитесь, что у вас есть `ПРИВАТНЫЙ_КЛЮЧ_СЕРВЕРА`, `ПУБЛИЧНЫЙ_КЛЮЧ_СЕРВЕРА`, `PRESHARED_КЛЮЧ_СЕРВЕРА` и адрес сервера (`vpn.example.com:51443`).

---

## 🧩 Пошаговая настройка

### 1️⃣ Создание интерфейса WireGuard

> **Используйте свой приватный ключ и имя интерфейса из вашей конфигурации!**

**Через WinBox**:
1. В меню слева выберите **Interfaces**.
2. Нажмите **+** и выберите **WireGuard**.
3. В открывшемся окне:
   - **Name**: Введите `wg-vpn`.
   - **MTU**: Установите `1420`.
   - **Private Key**: Вставьте `ПРИВАТНЫЙ_КЛЮЧ_СЕРВЕРА`.
4. Нажмите **OK**.

![Скриншот: Создание интерфейса WireGuard в WinBox](https://github.com/user-attachments/assets/fd4c254e-8e45-4036-b99e-8e57509d0219)

**Через терминал**:
```mikrotik
/interface wireguard add name=wg-vpn mtu=1420 private-key="ПРИВАТНЫЙ_КЛЮЧ_СЕРВЕРА"
```

- **name**: Имя интерфейса (`wg-vpn`).
- **mtu**: Размер пакета (1420 — стандарт для WireGuard).
- **private-key**: Приватный ключ роутера.

---

### 2️⃣ Назначение IP-адреса интерфейсу

> **Используйте свой IP-адрес и интерфейс из вашей конфигурации!**

**Через WinBox**:
1. В меню слева выберите **IP** → **Addresses**.
2. Нажмите **+** для добавления нового адреса.
3. В открывшемся окне:
   - **Address**: Введите `172.29.8.7/24`.
   - **Interface**: Выберите `wg-vpn`.
4. Нажмите **OK**.

![Скриншот: Добавление IP-адреса для wg-vpn](https://github.com/user-attachments/assets/8e748ec5-2fb4-4f79-a365-af32c63f4631)

**Через терминал**:
```mikrotik
/ip address add address=172.29.8.7/24 interface=wg-vpn
```

- **address**: IP-адрес для VPN.
- **interface**: Интерфейс (`wg-vpn`).

---

### 3️⃣ Настройка подключения к серверу (peer)

> **Используйте свои ключи, endpoint, allowed-address и параметры из вашей конфигурации и свои сети!**

**Через WinBox**:
1. В меню слева выберите **Interfaces** → **WireGuard** → вкладка **Peers**.
2. Нажмите **+** для добавления нового peer.
3. В открывшемся окне:
  - **Interface**: Выберите `wg-vpn`.
  - **Public Key**: Вставьте `ПУБЛИЧНЫЙ_КЛЮЧ_СЕРВЕРА`.
  - **Preshared Key**: Вставьте `PRESHARED_КЛЮЧ_СЕРВЕРА`.
  - **Endpoint**: Введите `vpn.example.com`.
  - **Endpoint Port**: Введите `51443`.
  - **Allowed IPs**: Вставьте: `0.0.0.0/0`
  - **Persistent Keepalive**: Установите `30`.
4. Нажмите **OK**.
   
![Скриншот: Настройка WireGuard Peer в WinBox](https://github.com/user-attachments/assets/cf852080-e7ee-4d59-86e7-75067ec71f0e)

**Через терминал**:
```mikrotik
/interface wireguard peers add \
   interface=wg-vpn \
   public-key="ПУБЛИЧНЫЙ_КЛЮЧ_СЕРВЕРА" \
   preshared-key="PRESHARED_КЛЮЧ_СЕРВЕРА" \
   endpoint-address=vpn.example.com \
   endpoint-port=51443 \
   allowed-address=0.0.0.0/0 \
   persistent-keepalive=30s
```

- **interface**: Интерфейс WireGuard.
- **public-key**: Публичный ключ сервера.
- **preshared-key**: Общий секрет.
- **endpoint-address/port**: Адрес и порт сервера.
- **allowed-address**: Сеть 0.0.0.0/0 укажим через файл ips.txt далее. 
- **persistent-keepalive**: Поддержка соединения (30 секунд).


### 4️⃣ Настройка NAT (маскарадинг)

> **Используйте свои параметры интерфейса и сети!**

**Через WinBox**:
1. В меню слева выберите **IP** → **Firewall** → вкладка **NAT**.
2. Нажмите **+** для добавления правила.
3. В открывшемся окне:
   - **Chain**: Выберите `srcnat`.
   - **Out. Interface**: Выберите `wg-vpn`.
   - **Action**: Выберите `masquerade`.
   - **Comment**: Введите `Masquerade VPN`.
4. Нажмите **OK**.
   
![Скриншот: Настройка NAT в WinBox](https://github.com/user-attachments/assets/56e00be0-d63b-4612-aec0-a0fb5fb55f94)

**Через терминал**:
```mikrotik
/ip firewall nat add chain=srcnat action=masquerade out-interface=wg-vpn comment="Masquerade VPN"
```

- **chain**: Цепочка NAT (`srcnat`).
- **action**: Подмена адреса (`masquerade`).
- **out-interface**: Интерфейс (`wg-vpn`).

---

### 5️⃣ Перенаправление DNS-запросов

> **Используйте свои локальные сети и параметры!**

**Через WinBox**:
1. В меню слева выберите **IP** → **Firewall** → вкладка **Mangle**.
2. Нажмите **+** для добавления правила.
3. В открывшемся окне:
   - **Chain**: Выберите `postrouting`.
   - **Src. Address**: Введите `192.168.88.0/24`.
   - **Out. Interface**: Выберите `wg-vpn`.
   - **Action**: Выберите `add src. to address list`.
   - **Address List**: Введите `RedirectDNS`.
   - **Address List Timeout**: Установите `60s`.
4. Нажмите **OK**.

![Скриншот: Настройка Mangle для DNS в WinBox](https://github.com/user-attachments/assets/2c993b31-98ca-4bbe-8301-7eb4a94ead85)

**Через терминал**:
```mikrotik
/ip firewall mangle add chain=postrouting src-address=192.168.88.0/24 action=add-src-to-address-list address-list=RedirectDNS address-list-timeout=60s out-interface=wg-vpn

```

- **src-address**: Локальная сеть.
- **action**: Добавление в список `RedirectDNS`.
- **out-interface**: Интерфейс (`wg-vpn`).

---

### 6️⃣ Скрипт автоматизации DNS, NAT И Добавление маршрутов

> ⚠️ **Обязательный шаг!**  
> Необходимо взять файл `/root/antizapret/result/ips.txt` с сервера AntiZapret и скопировать его в папку `files` на MikroTik.  
> В третьей строке скрипта Up обязательно укажите свой шлюз — в зависимости от выбранной при установке AntiZapret подсети (`172.29.8.1` или `10.29.8.1`).


**🧑‍💻 Автоматизация через Netwatch (WG-Monitor)**

**Через WinBox**:

1. В меню слева выберите **Tools → Netwatch**.
2. Нажмите **+** для добавления нового мониторинга.
3. В открывшемся окне:
  - **🌐 Host**: Введите `172.29.8.1` (IP-адрес за WireGuard-туннелем).
  - **⏱️ Interval**: Установите `00:00:15` (проверка каждые 15 секунд).
  - **⌛ Timeout**: Установите `5.00` (ожидание ответа 5 секунд).
  - **✅ Enabled**: Поставьте галочку, чтобы включить мониторинг.
  - **🟢 Up Script**: Вставьте код:
  
```mikrotik
/log info "[WG-Monitor] WireGuard is RUNNING — applying rules"
:local routeFile "ips.txt"
:local routeGateway "172.29.8.1"
:local routeComment "wg-auto-route"
/ip dns cache flush
:if ([:len [/ip firewall nat find comment="Redirect to Router"]] > 0) do={
  /ip firewall nat remove [find comment="Redirect to Router"]
}
/ip firewall nat add action=redirect chain=dstnat src-address-list="RedirectDNS" dst-port=53 protocol=udp comment="Redirect to Router"
/ip dns set servers=$routeGateway
:local content [/file get $routeFile contents]
:if ([:len $content] > 0) do={
    :local line ""
    :for i from=0 to=([:len $content] - 1) do={
        :local char [:pick $content $i]
        :if ($char != "\r" && $char != "\n") do={
            :set line ($line . $char)
        } else={
            :if ([:len $line] > 5) do={
                :do { /ip route add dst-address=$line gateway=$routeGateway distance=1 comment=$routeComment } on-error={}
            }
            :set line ""
        }
    }
    :if ([:len $line] > 5) do={
         :do { /ip route add dst-address=$line gateway=$routeGateway distance=1 comment=$routeComment } on-error={}
    }
}
:local dotPos [:find $routeGateway "."]
:if ($dotPos != "") do={
    :local firstOctet [:pick $routeGateway 0 $dotPos]
    :local derivedDstAddress ($firstOctet . ".30.0.0/15")
    :do { /ip route add dst-address=$derivedDstAddress gateway=$routeGateway distance=1 comment=$routeComment } on-error={}
}

```
  - **🔴 Down Script**: Вставьте код:
```mikrotik
/log info "[WG-Monitor] WireGuard is NOT running — reverting rules"
/ip firewall nat remove [find comment="Redirect to Router"]
/ip dns set servers=8.8.8.8
/ip dns cache flush
/ip route remove [find comment="wg-auto-route"]

```
1. Нажмите **OK**.

![Скриншот: Создание скрипта WG-Monitor в WinBox](https://github.com/user-attachments/assets/d598b9f6-e271-4580-83b2-48e97edad107)

---

**Через терминал**:

```mikrotik
/tool netwatch add host=172.29.8.1 interval=15s timeout=5s disabled=no \
up-script={:log info "[WG-Monitor] WireGuard is RUNNING — applying rules"; \
:local routeFile "ips.txt"; \
:local routeGateway "172.29.8.1"; \
:local routeComment "wg-auto-route"; \
/ip dns cache flush; \
:if ([:len [/ip firewall nat find comment="Redirect to Router"]] > 0) do={ \
  /ip firewall nat remove [find comment="Redirect to Router"]; \
}; \
/ip firewall nat add action=redirect chain=dstnat src-address-list="RedirectDNS" dst-port=53 protocol=udp comment="Redirect to Router"; \
/ip dns set servers=\$routeGateway; \
:local content [/file get \$routeFile contents]; \
:if ([:len \$content] > 0) do={ \
    :local line ""; \
    :for i from=0 to=([:len \$content] - 1) do={ \
        :local char [:pick \$content \$i]; \
        :if (\$char != "\r" && \$char != "\n") do={ \
            :set line (\$line . \$char); \
        } else={ \
            :if ([:len \$line] > 5) do={ \
                :do { /ip route add dst-address=\$line gateway=\$routeGateway distance=1 comment=\$routeComment } on-error={}; \
            }; \
            :set line ""; \
        }; \
    }; \
    :if ([:len \$line] > 5) do={ \
         :do { /ip route add dst-address=\$line gateway=\$routeGateway distance=1 comment=\$routeComment } on-error={}; \
    }; \
}; \
:local dotPos [:find \$routeGateway "."]; \
:if (\$dotPos != "") do={ \
    :local firstOctet [:pick \$routeGateway 0 \$dotPos]; \
    :local derivedDstAddress (\$firstOctet . ".30.0.0/15"); \
    :do { /ip route add dst-address=\$derivedDstAddress gateway=\$routeGateway distance=1 comment=\$routeComment } on-error={}; \
}} \
down-script={:log info "[WG-Monitor] WireGuard is NOT running — reverting rules"; \
/ip firewall nat remove [find comment="Redirect to Router"]; \
/ip dns set servers=8.8.8.8; \
/ip dns cache flush; \
/ip route remove [find comment="wg-auto-route"]}
```

---

**🔎 Что делает WG-Monitor:**
- 🟢 **WireGuard работает:** перенаправляет DNS через VPN, отключает DNS провайдера, обновляет NAT.
- 🔴 **WireGuard не работает:** возвращает стандартные настройки DNS и NAT.

---
### 7️⃣ Настройка DHCP-клиента

DHCP-клиент на основном интерфейсе (например, ether1) позволяет роутеру получать IP-адрес и другие параметры от провайдера.

- **WinBox**:  
  Перейдите в **IP → DHCP Client → Add (+)**.  
  Выберите интерфейс (например, ether1), уберите галочку "Use Peer DNS", поставьте галочку "Add Default Route".
- **Терминал**:
  ```bash
  /ip dhcp-client add interface=ether1 use-peer-dns=no add-default-route=yes
  ```

> - `use-peer-dns=no` — отключает получение DNS от провайдера, чтобы использовать свои настройки
> - `add-default-route=yes` — добавляет маршрут по умолчанию для выхода в интернет

> **Проверьте, что интерфейс ether1 подключён к интернету и получает IP-адрес.**

---

**💡 Пример: DHCP Client Add**

![DHCP Client Add](https://github.com/Kirito0098/AntiZapret-OpenVPN-Mikrotik/raw/main/screenshot/WinBox_WfS4CCVDPU.png)
*IP → DHCP Client → Add*

---

### 8️⃣ Отключение FastTrack

**Через WinBox**:
1. В меню слева выберите **IP** → **Firewall** → вкладка **Filter Rules**.
2. Найдите правило с `Action = fasttrack-connection`.
3. Выделите его и нажмите **Disable** (иконка с крестиком).

![Скриншот: Отключение FastTrack в WinBox](https://github.com/user-attachments/assets/33e5b174-c298-48a1-b103-f35510d203b3)

**Через терминал**:
```mikrotik
/ip firewall filter disable [find action=fasttrack-connection]
```
---
### 9️⃣ Важно про MTU и TCPMSS

> 🛑 **Если сайты открываются медленно, соединения обрываются или не работает TLS 1.3 — вероятная причина — некорректный MTU или MSS!**  
> ⚡ **Рекомендуется всегда настраивать автоматическое ограничение MSS для WireGuard, но достаточно сделать это только на одной стороне — сервере или MikroTik!**

Для стабильной работы TCP через WireGuard настройте автоматическое ограничение MSS (Maximum Segment Size) — **на сервере или на MikroTik** (выберите один вариант).

**Проверьте наличие правила TCPMSS:**
```bash
sudo iptables -t mangle -S | grep TCPMSS
```
> **ℹ️ Если правило уже присутствует — ничего добавлять не нужно!**  
> Проверьте наличие через команду выше. Если оно есть, переходите к следующему шагу.

#### 🐧 На сервере WireGuard (Linux):

```bash
iptables -t mangle -A FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
```

> ⚠️ Это правило временное: после перезагрузки сервера оно исчезнет.  
> Чтобы правило сохранялось, добавьте его в скрипт автозагрузки (например, `/etc/rc.local`) или используйте persistent iptables (например, пакет `iptables-persistent`).
>  
> 🔧 Это правило автоматически подстраивает MSS под реальный MTU маршрута, предотвращая "тихие" потери пакетов.

#### 🛡️ На MikroTik:

**Через WinBox**:
1. 🖥️ Откройте меню **IP** → **Firewall** → вкладка **Mangle**.
2. ➕ Нажмите **+** для добавления нового правила.
3. ⚙️ В открывшемся окне:
  - **Chain**: выберите `forward`.
  - **Protocol**: выберите `tcp`.
  - **TCP Flags**: выберите `syn`.
  - **Out. Interface**: выберите `wg-vpn`.
  - **Action**: выберите `change MSS`.
  - **New MSS**: выберите `clamp to pmtu`.
  - **Comment**: введите `Clamp MSS for WG`.
4. ✅ Нажмите **OK**.

**Через Терминал:**
```mikrotik
/ip firewall mangle add chain=forward protocol=tcp tcp-flags=syn action=change-mss new-mss=clamp-to-pmtu out-interface=wg-vpn comment="Clamp MSS for WG"
```

## ➕ Добавление новых подсетей

> **Добавляйте только свои подсети, используйте свои значения!**

1. **Добавить маршруты для новых подсетей:**

Добавляет маршруты для новых подсетей через VPN-шлюз.

```shell
/ip route add dst-address=НОВАЯ_ПОДСЕТЬ_1 gateway=172.29.8.1 distance=1
/ip route add dst-address=НОВАЯ_ПОДСЕТЬ_2 gateway=172.29.8.1 distance=1
```

---

## ✅ Проверка работы

- **Пинг VPN-шлюза** 🏓:
  Проверяет доступность VPN-шлюза через WireGuard-интерфейс.

  ```shell
  /ping 172.29.8.1 interface=wg-vpn
  ```

- **Трассировка маршрута** 🧭:
  Проверяет маршрут до внешнего адреса через VPN.

  ```shell
  /tool traceroute 104.16.0.1
  ```

- **Просмотр трафика на VPN-интерфейсе** 📊:
  Быстрый просмотр трафика на VPN-интерфейсе по порту 443.

  ```shell
  /tool sniffer quick interface=wg-vpn port=443
  ```
- **Тестирование** 🧪:
   ```shell
   /interface wireguard peers print
   ```
- **Логи** 📋:
   ```shell
   /log print
   ```

---

## ❓ FAQ

**Q: Как узнать, что WireGuard работает?**  
A: Проверьте статус интерфейса и peer'а через `/interface wireguard peers print`.

**Q: Как добавить новую подсеть?**  
A: Добавьте её в allowed-address peer'а и создайте маршрут.

**Q: Как проверить, что DNS перенаправляется через VPN?**  
A: Проверьте правило NAT с комментарием "Redirect to Router".

---

## 🛠️ Troubleshooting

### 1. Проблемы с MTU

- **Симптомы:** сайты открываются медленно, не загружаются, обрывы соединения.
- **Решение:** уменьшите MTU интерфейса WireGuard до 1380 или ниже:
  ```shell
  /interface wireguard set [find name=wg-vpn] mtu=1380
  ```

### 2. Проблемы с DNS

- **Симптомы:** сайты не открываются, нет доступа к интернету через VPN.
- **Решение:**
  - Проверьте, что в настройках DNS указаны корректные адреса (например, 172.29.8.1).
  - Очистите DNS-кэш:
    ```shell
    /ip dns cache flush
    ```
  - Проверьте, что правило перенаправления DNS работает:
    ```shell
    /ip firewall nat print where comment="Redirect to Router"
    ```

### 3. Проблемы с маршрутизацией

- **Симптомы:** нет доступа к нужным подсетям, VPN-трафик не проходит.
- **Решение:**
  - Проверьте, что все нужные маршруты добавлены:
    ```shell
    /ip route print
    ```
  - Проверьте, что шлюз (gateway) доступен:
    ```shell
    /ping 172.29.8.1 interface=wg-vpn
    ```
  - Убедитесь, что подсети указаны в allowed-address peer'а.

### 4. Проблемы с ключами

- **Симптомы:** VPN не подключается, peer не появляется в списке активных.
- **Решение:**
  - Проверьте правильность приватного и публичного ключей.
  - Проверьте, что preshared-key совпадает на обеих сторонах.

### 5. FastTrack не отключён

- **Симптомы:** VPN работает нестабильно, трафик не маршрутизируется.
- **Решение:** отключите FastTrack:
  ```shell
  /ip firewall filter disable [find action=fasttrack-connection]
  ```

### 6. Нет связи с сервером WireGuard

- **Симптомы:** peer не устанавливает соединение, нет handshake.
- **Решение:**
  - Проверьте правильность endpoint-address и port.
  - Проверьте, что сервер WireGuard доступен с роутера (ping, traceroute).
  - Проверьте, что firewall не блокирует UDP-порт WireGuard (51443).

---

## 📚 Полезные ссылки
- [Официальная Wiki MikroTik (WireGuard)](https://help.mikrotik.com/docs/display/ROS/WireGuard)
- [Официальная документация WireGuard](https://www.wireguard.com/)
- [MikroTik RouterOS Wiki](https://wiki.mikrotik.com/wiki/Main_Page)
- [WireGuard Quick Start](https://www.wireguard.com/quickstart/)



