# Конфигурирование интерфейсов

> Кратко: имена интерфейсов, просмотр текущих адресов, назначение адреса, DHCP-клиент, резолвер, современные инструменты (Netplan, NetworkManager/nmcli) и файл статистики.

***

### a. Имена интерфейсов (`enp0s3`, `eth0` и т.п.)

* Два основных подхода:
  * **predictable names** (systemd/udev): `enp0s3`, `ens33`, `wlp2s0` — основаны на шине/порте.
  * **legacy**: `eth0`, `eth1` — старые, могут появляться после отключения predictable naming.
* Посмотреть:

```bash
ip link show
# или
ls /sys/class/net
```

* Переименование: через udev rule или netplan/NetworkManager конфигурацию. Не менять спонтанно в продакшн.

***

### b. `ifconfig` (получение информации о адресах)

* Устаревший, но часто установлен:

```bash
ifconfig -a           # показать все интерфейсы и адреса
ifconfig eth0 up
ifconfig eth0 192.168.1.10 netmask 255.255.255.0
ifconfig eth0 down
```

* Вместо него предпочтительнее `ip` (ниже). Используйте `ifconfig` только если он доступен.

***

### c. `ip` (контексты `link`, `address`, назначение, установка адреса)

* Современный инструмент (iproute2). Ключевые команды:

```bash
# Показать интерфейсы и link-level статус
ip link show

# Включить/выключить интерфейс
ip link set dev eth0 up
ip link set dev eth0 down

# Показать адреса (IPv4 и IPv6)
ip addr show
ip -4 addr show eth0
ip -6 addr show eth0

# Добавить/удалить адрес
ip addr add 192.168.1.10/24 dev eth0
ip addr del 192.168.1.10/24 dev eth0

# Добавить маршрут по умолчанию
ip route add default via 192.168.1.1 dev eth0

# Получить путь до хоста (какой интерфейс/next-hop)
ip route get 8.8.8.8
```

* Контексты: `ip link` (L1/L2), `ip addr` (L3 адреса), `ip route` (маршруты).

***

### d. `dhclient`

* DHCP-клиент (ISC). Примеры:

```bash
# Запрос адреса на интерфейсе
dhclient -v eth0

# Освободить аренду
dhclient -r eth0

# Запустить в фоне (daemon) обычно systemd/NetworkManager управляет этим
```

* Альтернативы: `dhcpcd`, `systemd-networkd` (через netplan), NetworkManager.

***

### e. `/etc/resolv.conf` — назначение и синтаксис

* Назначение: список резолверов DNS и опций для локального resolver.
* Формат:

```
# пример /etc/resolv.conf
nameserver 8.8.8.8
nameserver 8.8.4.4
search example.local corp.example
options timeout:2 attempts:3 rotate
```

* Примечания:
  * Файл часто управляется NetworkManager, systemd-resolved или dhclient. Не редактируйте напрямую, если управление автоматизировано.
  * `nameserver` — IP резолвера. Можно указать несколько строк.
  * `search` — суффиксы для резолвинга коротких имён.
  * `options` — тонкая настройка (timeout, attempts, rotate и т.д.).

***

### f. Netplan (возможности, конфигурационные файлы, принцип работы)

* Ubuntu/дистры: `/etc/netplan/*.yaml`. Netplan генерирует конфигурации для `systemd-networkd` или `NetworkManager`.
* Основные команды:

```bash
# проверить и применить
netplan try        # применяет временно; откатит при отказе
netplan apply      # применяет окончательно
netplan generate   # сгенерировать backend-конфиг
```

* Пример DHCP (ethernet):

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true
```

* Пример статического адреса + шлюз + DNS:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      addresses: [192.168.10.10/24]
      gateway4: 192.168.10.1
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]
        search: [example.local]
```

* Поддерживает: VLAN, bridges, bonds, tunnels, multiple renderers. YAML строго по отступам (две пробелы). После правки — `netplan apply`.

***

### g. `nmcli` (возможности, принцип работы)

* CLI для NetworkManager. Управляет соединениями, device, Wi-Fi, VPN.
* Полезные команды:

```bash
nmcli device status
nmcli connection show
nmcli connection show "Wired connection 1"
# Добавить статическую конфигурацию
nmcli connection add type ethernet ifname enp0s3 con-name office ipv4.addresses 192.168.10.10/24 ipv4.gateway 192.168.10.1 ipv4.dns 8.8.8.8 ipv4.method manual
# Включить/выключить connection
nmcli connection up office
nmcli connection down office
# Быстрая правка существующего профиля
nmcli connection modify office ipv4.dns "8.8.8.8 8.8.4.4"
```

* NetworkManager автоматически обновляет `/etc/resolv.conf` (если настроен), управляет DHCP, Wi-Fi и VPN; nmcli — scriptable интерфейс.

***

### h. Файл статистики интерфейсов `/proc/net/dev`

* Формат: первая строка заголовки, далее строки по интерфейсам.
* Пример вывода:

```
Inter-|   Receive                                                |  Transmit
 face |bytes    packets errs drop fifo frame compressed multicast|bytes    packets errs drop fifo colls carrier compressed
  eth0: 12345678  12345    0    2    0     0          0       0 87654321  54321    0    0    0     0       0          0
```

* Поля слева — **receive**: `bytes packets errs drop ... multicast`. Справа — **transmit**: `bytes packets errs drop ... carrier`.
* Быстрый парсинг:

```bash
# показать rx/tx bytes для eth0
awk '/eth0:/ {print "rx_bytes="$2,"tx_bytes="$10}' /proc/net/dev
# или суммарно
cat /proc/net/dev
```

* Используют для мониторинга throughput, ошибок, drop-ов. Подсчёт скорости: взять два замера и разделить по времени.

***

### Доп. полезные утилиты и заметки

* `ethtool eth0` — показать link speed, duplex, autoneg, offload.
* `ip -s link` — статистика интерфейса (packets/bytes/errors).
* `ss -tulpen` / `netstat -tulpen` — проверить слушающие порты после настройки IP.
* При автоматизации используйте systemd-networkd/Netplan/NM в зависимости от окружения. Не смешивайте конфиг-файлы разных менеджеров без явного понимания, кто управляет `/etc/resolv.conf`.

***

### Короткая шпаргалка команд

```bash
# посмотреть интерфейсы
ip link show
ip addr show

# поднять интерфейс и назначить IP
ip link set dev enp0s3 up
ip addr add 192.168.10.10/24 dev enp0s3

# запрос DHCP
dhclient -v enp0s3

# nmcli быстрые операции
nmcli device status
nmcli connection up <name>

# netplan
netplan apply
```

***

Вопросы для самопроверки

1. Чем `ip addr add 192.168.1.10/24 dev eth0` отличается от `ifconfig eth0 192.168.1.10 netmask 255.255.255.0`?
2. Где хранится конфигурация Netplan? Какая команда применяет её?
3. Какие поля важны в `/proc/net/dev` для определения потерь пакетов?
