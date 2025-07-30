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
      - [**Метод №1**](#метод-1)
      - [**Метод №2**](#метод-2)
    - [4️⃣ Настройка NAT (маскарадинг)](#4️⃣-настройка-nat-маскарадинг)
    - [5️⃣ Добавление маршрутов](#5️⃣-добавление-маршрутов)
    - [6️⃣ Перенаправление DNS-запросов](#6️⃣-перенаправление-dns-запросов)
    - [7️⃣ Скрипт автоматизации DNS и NAT](#7️⃣-скрипт-автоматизации-dns-и-nat)
    - [8️⃣ Настройка планировщика](#8️⃣-настройка-планировщика)
    - [9️⃣ Отключение FastTrack](#9️⃣-отключение-fasttrack)
    - [🔟 Важно про MTU и TCPMSS](#-важно-про-mtu-и-tcpmss)
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
AllowedIPs = 172.29.8.0/24,172.30.0.0/15,103.21.244.0/22,103.22.200.0/22,103.31.4.0/22,104.16.0.0/12,
104.24.0.0/14,108.162.192.0/18,131.0.72.0/22,141.101.64.0/18,162.158.0.0/15,172.64.0.0/13,173.245.48.0/20,
188.114.96.0/20,190.93.240.0/20,197.234.240.0/22,198.41.128.0/17,3.74.0.0/15,34.0.0.0/16,34.1.224.0/19,
35.207.0.0/16,35.212.0.0/14,35.217.0.0/18,35.219.224.0/19,54.193.0.0/16,66.22.192.0/18
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

**Если у вас много сетей, роутер может уйти в цикличную перезагрузку — используйте Метод №2**

#### **Метод №1**

**Через WinBox**:
1. В меню слева выберите **Interfaces** → **WireGuard** → вкладка **Peers**.
2. Нажмите **+** для добавления нового peer.
3. В открывшемся окне:
  - **Interface**: Выберите `wg-vpn`.
  - **Public Key**: Вставьте `ПУБЛИЧНЫЙ_КЛЮЧ_СЕРВЕРА`.
  - **Preshared Key**: Вставьте `PRESHARED_КЛЮЧ_СЕРВЕРА`.
  - **Endpoint**: Введите `vpn.example.com`.
  - **Endpoint Port**: Введите `51443`.
  - **Allowed IPs**: Вставьте:
    ```
    172.29.8.0/24,172.30.0.0/15,103.21.244.0/22,103.22.200.0/22,103.31.4.0/22,104.16.0.0/12,104.24.0.0/14,108.162.192.0/18,131.0.72.0/22,141.101.64.0/18,162.158.0.0/15,172.64.0.0/13,173.245.48.0/20,188.114.96.0/20,190.93.240.0/20,197.234.240.0/22,198.41.128.0/17,3.74.0.0/15,34.0.0.0/16,34.1.224.0/19,35.207.0.0/16,35.212.0.0/14,35.217.0.0/18,35.219.224.0/19,54.193.0.0/16,66.22.192.0/18
    ```
  - **Persistent Keepalive**: Установите `15`.
4. Нажмите **OK**.
   
![Скриншот: Настройка WireGuard Peer в WinBox](https://github.com/user-attachments/assets/c64ddee9-cf57-4990-bebe-3e92bb4cfc20)

**Через терминал**:
```mikrotik
/interface wireguard peers add \
   interface=wg-vpn \
   public-key="ПУБЛИЧНЫЙ_КЛЮЧ_СЕРВЕРА" \
   preshared-key="PRESHARED_КЛЮЧ_СЕРВЕРА" \
   endpoint-address=vpn.example.com \
   endpoint-port=51443 \
   allowed-address=172.29.8.0/24,172.30.0.0/15,103.21.244.0/22,103.22.200.0/22,103.31.4.0/22,104.16.0.0/12,104.24.0.0/14,108.162.192.0/18,131.0.72.0/22,141.101.64.0/18,162.158.0.0/15,172.64.0.0/13,173.245.48.0/20,188.114.96.0/20,190.93.240.0/20,197.234.240.0/22,198.41.128.0/17,3.74.0.0/15,34.0.0.0/16,34.1.224.0/19,35.207.0.0/16,35.212.0.0/14,35.217.0.0/18,35.219.224.0/19,54.193.0.0/16,66.22.192.0/18 \
   persistent-keepalive=15s
```

- **interface**: Интерфейс WireGuard.
- **public-key**: Публичный ключ сервера.
- **preshared-key**: Общий секрет.
- **endpoint-address/port**: Адрес и порт сервера.
- **allowed-address**: Сети, доступные через VPN.
- **persistent-keepalive**: Поддержка соединения (15 секунд).

#### **Метод №2**

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
- **allowed-address**: Сети, доступные через VPN.
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

### 5️⃣ Добавление маршрутов

> **Добавляйте маршруты только для своих сетей из вашей конфигурации!**

**Через WinBox**:
1. В меню слева выберите **IP** → **Routes**.
2. Нажмите **+** для добавления маршрута.
3. Для каждой подсети:
   - **Dst. Address**: Введите подсеть (например, `172.29.8.0/24`).
   - **Gateway**: Введите `172.29.8.1`.
   - **Distance**: Установите `1`.
   - **Check Gateway**: Выберите `ping`.
4. Повторите для всех подсетей (Используйте ваши подсети):
   Пример
   ```
   172.29.8.0/24
   172.30.0.0/15
   103.21.244.0/22
   103.22.200.0/22
   103.31.4.0/22
   104.16.0.0/12
   104.24.0.0/14
   108.162.192.0/18
   131.0.72.0/22
   141.101.64.0/18
   162.158.0.0/15
   172.64.0.0/13
   173.245.48.0/20
   188.114.96.0/20
   190.93.240.0/20
   197.234.240.0/22
   198.41.128.0/17
   3.74.0.0/15
   34.0.0.0/16
   34.1.224.0/19
   35.207.0.0/16
   35.212.0.0/14
   35.217.0.0/18
   35.219.224.0/19
   54.193.0.0/16
   66.22.192.0/18
   ```
5. Нажмите **OK** для каждого маршрута.
 ![Скриншот: Добавление маршрутов в WinBox](https://github.com/user-attachments/assets/f45b238e-f961-45ed-a525-b825ac96f9eb)

**Через терминал (Используйте ваши подсети)**:
```mikrotik
/ip route add dst-address=172.29.8.0/24 gateway=172.29.8.1 distance=1
/ip route add dst-address=172.30.0.0/15 gateway=172.29.8.1 distance=1
/ip route add dst-address=103.21.244.0/22 gateway=172.29.8.1 distance=1
/ip route add dst-address=103.22.200.0/22 gateway=172.29.8.1 distance=1
/ip route add dst-address=103.31.4.0/22 gateway=172.29.8.1 distance=1
/ip route add dst-address=104.16.0.0/12 gateway=172.29.8.1 distance=1
/ip route add dst-address=104.24.0.0/14 gateway=172.29.8.1 distance=1
/ip route add dst-address=108.162.192.0/18 gateway=172.29.8.1 distance=1
/ip route add dst-address=131.0.72.0/22 gateway=172.29.8.1 distance=1
/ip route add dst-address=141.101.64.0/18 gateway=172.29.8.1 distance=1
/ip route add dst-address=162.158.0.0/15 gateway=172.29.8.1 distance=1
/ip route add dst-address=172.64.0.0/13 gateway=172.29.8.1 distance=1
/ip route add dst-address=173.245.48.0/20 gateway=172.29.8.1 distance=1
/ip route add dst-address=188.114.96.0/20 gateway=172.29.8.1 distance=1
/ip route add dst-address=190.93.240.0/20 gateway=172.29.8.1 distance=1
/ip route add dst-address=197.234.240.0/22 gateway=172.29.8.1 distance=1
/ip route add dst-address=198.41.128.0/17 gateway=172.29.8.1 distance=1
/ip route add dst-address=3.74.0.0/15 gateway=172.29.8.1 distance=1
/ip route add dst-address=34.0.0.0/16 gateway=172.29.8.1 distance=1
/ip route add dst-address=34.1.224.0/19 gateway=172.29.8.1 distance=1
/ip route add dst-address=35.207.0.0/16 gateway=172.29.8.1 distance=1
/ip route add dst-address=35.212.0.0/14 gateway=172.29.8.1 distance=1
/ip route add dst-address=35.217.0.0/18 gateway=172.29.8.1 distance=1
/ip route add dst-address=35.219.224.0/19 gateway=172.29.8.1 distance=1
/ip route add dst-address=54.193.0.0/16 gateway=172.29.8.1 distance=1
/ip route add dst-address=66.22.192.0/18 gateway=172.29.8.1 distance=1

```

- **dst-address**: Подсеть для маршрута.
- **gateway**: Шлюз (`172.29.8.1`).
- **check-gateway**: Проверка шлюза (`ping`).

---

### 6️⃣ Перенаправление DNS-запросов

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

### 7️⃣ Скрипт автоматизации DNS и NAT

> **Проверьте, что параметры скрипта соответствуют вашей конфигурации!**

**Через WinBox**:
1. В меню слева выберите **System** → **Scripts**.
2. Нажмите **+** для создания скрипта.
3. В открывшемся окне:
   - **Name**: Введите `WG-Monitor`.
   - **Policy**: Установите галочки `read`, `write`, `test`.
   - **Source**: Вставьте код:
    ```mikrotik
    :local iface "wg-vpn"
    :local isRunning [/interface get [find name="wg-vpn"] running]
    :global wgLastState

    :if ([:typeof $wgLastState] = "nothing" || $wgLastState != $isRunning) do={
      :if ($isRunning) do={
        /log info "[WG-Monitor] WireGuard is RUNNING, applying rules"
        /ip dhcp-client set [find interface="ether1"] use-peer-dns=no
        /ip dns cache flush
        /ip firewall nat remove [find comment="Redirect to Router"]
        /ip firewall nat add action=redirect chain=dstnat src-address-list="RedirectDNS" dst-port=53,5353,1253 protocol=udp comment="Redirect to Router"
        /ip dns set servers=172.29.8.1
      } else={
        /log info "[WG-Monitor] WireGuard is NOT running, reverting rules"
        /ip firewall nat remove [find comment="Redirect to Router"]
        /ip dns set servers=""
        /ip dhcp-client set [find interface="ether1"] use-peer-dns=yes
        /ip dns cache flush
      }
      :set wgLastState $isRunning
    }
    ```
4. Нажмите **OK**.

![Скриншот: Создание скрипта WG-Monitor в WinBox](https://github.com/user-attachments/assets/267bd173-0c2c-4cbc-b9ba-9ab9a3c14cba)

**Через терминал**:
```mikrotik
/system script
add name=WG-Monitor source={
:local iface "wg-vpn"
:local isRunning [/interface get [find name="wg-vpn"] running]
:global wgLastState

:if ([:typeof $wgLastState] = "nothing" || $wgLastState != $isRunning) do={
    :if ($isRunning) do={
        /log info "[WG-Monitor] WireGuard is RUNNING, applying rules"
        /ip dhcp-client set [find interface="ether1"] use-peer-dns=no
        /ip dns cache flush
        /ip firewall nat remove [find comment="Redirect to Router"]
        /ip firewall nat add action=redirect chain=dstnat src-address-list="RedirectDNS" dst-port=53,5353,1253 protocol=udp comment="Redirect to Router"
        /ip dns set servers=172.29.8.1
    } else={
        /log info "[WG-Monitor] WireGuard is NOT running, reverting rules"
        /ip firewall nat remove [find comment="Redirect to Router"]
        /ip dns set servers=""
        /ip dhcp-client set [find interface="ether1"] use-peer-dns=yes
        /ip dns cache flush
    }
    :set wgLastState $isRunning
}
}

```

**Что делает скрипт**:
- Проверяет состояние WireGuard.
- Если работает: перенаправляет DNS через VPN, отключает DNS провайдера.
- Если не работает: возвращает стандартные настройки DNS.

---

### 8️⃣ Настройка планировщика

**Через WinBox**:
1. В меню слева выберите **System** → **Scheduler**.
2. Нажмите **+** для добавления расписания.
3. В открывшемся окне:
   - **Name**: Введите `checkWG`.
   - **Interval**: Установите `00:00:10` (10 секунд).
   - **On Event**: Введите `:system script run WG-Monitor`.
   - **Policy**: Установите галочки `read`, `write`, `policy`, `test`.
4. Нажмите **OK**.

![Скриншот: Настройка планировщика в WinBox](https://github.com/user-attachments/assets/078b3f5f-662a-452f-9abc-76281444f126)

**Через терминал**:
```mikrotik
/system scheduler
add interval=10s name=checkWG on-event=":system script run WG-Monitor" policy=read,write,policy,test

```

- **interval**: Период проверки (10 секунд).
- **on-event**: Запуск скрипта `WG-Monitor`.

---

### 9️⃣ Отключение FastTrack

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
### 🔟 Важно про MTU и TCPMSS

> 🛑 **Если сайты открываются медленно, соединения обрываются или не работает TLS 1.3 — скорее всего, проблема в MTU или MSS!**  
> ⚡ **Рекомендуется всегда использовать эти правила для WireGuard, но достаточно настроить только на одной стороне — сервере или MikroTik!**

Для стабильной работы TCP через WireGuard настройте автоматическое ограничение MSS (Maximum Segment Size) **или на сервере, или на MikroTik** — достаточно одного варианта.

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



