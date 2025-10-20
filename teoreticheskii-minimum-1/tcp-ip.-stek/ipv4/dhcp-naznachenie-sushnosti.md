---
description: клиент, сервер, релей
---

# DHCP назначение, сущности

> DHCP автоматически выдаёт сетевые параметры хостам в локальной сети: IP, маску, шлюз, DNS и другие опции. Упрощает управление адресным пулом и аренду адресов.

### Сущности

* **DHCP-клиент** — запрашивает параметры (обычно хост/OS).
* **DHCP-сервер** — хранит пул(ы) адресов и выдаёт конфигурацию.
* **DHCP-relay (агент)** — пересылает DHCP-сообщения между клиентом и сервером через маршрутизатор/реве́йл (ip helper).
* **База аренды (leases)** — таблица выданных адресов с тайм-аутами.

### Ключевые сообщения (DORA и др.)

* **DHCPDISCOVER** — клиент ищет сервер.
* **DHCPOFFER** — сервер предлагает параметры.
* **DHCPREQUEST** — клиент запрашивает выбранное предложение.
* **DHCPACK** — сервер подтверждает аренду.
* **DHCPNAK** — отклонение (например, неверный адрес).
* **DHCPRELEASE** — клиент освобождает адрес.
* **DHCPINFORM** — клиент с ручным IP запрашивает только опции.

Абревиатура DORA = Discover → Offer → Request → Ack.

### Основные опции (часто используемые)

* `option subnet-mask` (1)
* `option routers` (3) — default gateway
* `option domain-name-servers` (6) — DNS
* `option domain-name` (15)
* `option lease-time` / `default-lease-time` / `max-lease-time` (51)
* `option server-identifier` (54)

### Поведение: пулы и аренда

* Сервер хранит диапазоны `range` и выдает адреса временно (lease\_seconds).
* Формула перевода lease в дни: $$\text{days}=\frac{\text{seconds}}{86400}.$$
* Клиент периодически обновляет аренду (renew) — обычно на 50% времени аренды происходит попытка продления.

### Relay / Forwarder

* Если клиент и сервер в разных L2, маршрутизатор/хост должен пересылать UDP порты 67/68.
* Частая команда Cisco: `ip helper-address <DHCP-server-IP>`.

### Пример минимальной конфигурации (isc-dhcp-server)

```
# /etc/dhcp/dhcpd.conf (пример)
default-lease-time 600;
max-lease-time 7200;
subnet 192.168.10.0 netmask 255.255.255.0 {
  range 192.168.10.100 192.168.10.200;
  option routers 192.168.10.1;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
  option domain-name "example.local";
}
```

### Команды и диагностика

*   Захват DHCP-трафика:

    ```bash
    tcpdump -n -i eth0 'port 67 or port 68'
    ```
* Проверить статус сервера: `systemctl status isc-dhcp-server` (или `dhcpd`).
* Просмотреть аренды: `/var/lib/dhcp/dhcpd.leases` или `/var/db/dhcpd.leases`.
* Клиент вручную: `dhclient -v eth0` / `dhcpcd` / `nmcli device connect`.

### Проблемы и безопасность

* **Rogue DHCP** — посторонний сервер раздаёт неверные маршруты/DNS.
* **DHCP starvation** — атака, истощающая пул адресов.
* Решения: DHCP snooping на коммутаторах, bind DHCP на trusted ports, фильтрация, rate-limits, авторизация (802.1X).

### Практические советы

* Делите пул на подсети и используйте резервации (static mappings) по MAC для серверов.
* Устанавливайте разумные `default/max lease` в зависимости от мобильности клиентов.
* В корпоративных сетях комбинируйте DHCP relay + централизованные серверы с HA (failover).

### Короткие экзаменационные тезисы

1. DHCP автоматизирует выдачу IP и опций; использует UDP порты 67(server)/68(client).
2. DORA = Discover, Offer, Request, Ack.
3. Relay пересылает DHCP между L2-доменами (`ip helper-address`).
4. Основные опции: mask, routers, dns, lease time.
5. Защита: DHCP snooping предотвращает rogue DHCP и starvation.

### Вопросы для самопроверки

1. Какую роль выполняет DHCP-relay и почему он нужен?
2. Что происходит, если клиент не продлевает аренду до истечения?
3. Как защитить сеть от rogue DHCP?
