# Архитектура стека, инкапсуляция. Назначение основных протоколов

## Архитектура стека TCP/IP — кратко

* Модель TCP/IP обычно представляют как 4 уровня:
  1. **Link (канальный/физический)** — Ethernet, Wi-Fi, PPP.
  2. **Internet (сетевой)** — IP.
  3. **Transport (транспортный)** — TCP, UDP.
  4. **Application (прикладной)** — HTTP, DNS, SSH и т.д.
* Соответствие OSI: Link ≈ L1–L2, Internet ≈ L3, Transport ≈ L4, Application ≈ L5–L7.

## Инкапсуляция (последовательность)

Приложение → транспортный заголовок → сетевой заголовок → канальный заголовок → биты по физике.

* Единицы данных: Application payload → **segment** (TCP) / **datagram** (UDP) → **packet** (IP) → **frame** (Ethernet).
* Схема:

```mermaid
flowchart LR
  App -->|payload| Trans[Transport: TCP/UDP]
  Trans -->|segment/datagram| Net[IP]
  Net -->|packet| Link[Ethernet frame]
  Link --> Phys[Физический уровень]
```

## Коротко о протоколах и их назначении

### IP (Internet Protocol)

* Роль: адресация и маршрутизация пакетов. Работа в режиме **best-effort** (без гарантии доставки).
* Функции: фрагментация/дефрагментация, TTL (time to live), поля протокола (указывает L4).
* IPv4: адреса 32-бит. Маршрутизация по префиксам.
* Диагностика: `ip route`, `traceroute` (TTL-based), `ip addr`.
* Wireshark: `ip.src == x.x.x.x` / `ip.dst == x.x.x.x`.

### TCP (Transmission Control Protocol)

* Роль: надёжный поток байт между сокетами. Механизмы: трехсторонний handshake (SYN/SYN-ACK/ACK), нумерация Seq/Ack, подтверждения, контроль перегрузки и окно (flow/congestion control), порядок и ретрансмиссии.
* Полезные поля: `SrcPort|DstPort|Seq|Ack|Flags(SYN,ACK,FIN,RST)|Window`.
* Портовый диапазон: \$$0..65535\$$; well-known \$$0..1023\$$.
* Диагностика: `netstat -tn`, `ss -t`, `tcpdump tcp` ; Wireshark: `tcp.stream eq N`.
* Поведение: устанавливает соединение, поддерживает порядок, обеспечивает надёжность.

### UDP (User Datagram Protocol)

* Роль: простой, ненадёжный, безустановочный транспорт. Меньшая задержка, нет управления перегрузкой/реконнектов. Используется для DNS, VoIP, потоков.
* Поля: `SrcPort|DstPort|Length|Checksum`.
* Диагностика: `netstat -un`, `ss -u`; Wireshark: `udp.port == 53`.

### ARP (Address Resolution Protocol)

* Роль: перевод IP → MAC в локальной L2-сети (ARP request / ARP reply).
* Gratuitous ARP используется для объявления/проверки IP→MAC.
* Проблемы: ARP-spoofing.
* Диагностика: `arp -a`, Wireshark: `arp`.

### ICMP (Internet Control Message Protocol)

* Роль: служебные сообщения для контроля и диагностики (destination unreachable, time exceeded, echo request/reply used by `ping`).
* Используется `traceroute` (TTL expired → ICMP Time Exceeded) и `ping` (Echo).
* Wireshark: `icmp`.

## Инкапсуляция полей (коротко)

* IP вносит поле `Protocol` (например, `6` для TCP, `17` для UDP).
* При получении стек проверяет IP→Protocol и отдаёт payload соответствующему транспортному протоколу.

## Диагностика — быстрый набор команд

* Проверка адресов/маршрутов: `ip addr`, `ip route`.
* Проверка ARP: `ip neigh` / `arp -a`.
* Проверка TCP/UDP сокетов: `ss -tuna`, `netstat -tulpen`.
* Ping/traceroute: `ping`, `traceroute` / `traceroute -I`(ICMP).
* Захват: `tcpdump -n -i <if> icmp or arp or tcp` ; Wireshark — удобный GUI.

## Важные экзаменационные тезисы

1. IP — ненадёжный, маршрутизирующий протокол с TTL и фрагментацией.
2. TCP — соединенческий, обеспечивает надёжность, порядок и контроль перегрузки.
3. UDP — безустановочный, низкая задержка, без гарантий доставки.
4. ARP — локальная сопоставительная служба IP→MAC.
5. ICMP — протокол диагностики и сообщений об ошибках.

## Вопросы для самопроверки

1. Чем TCP и UDP отличаются по ответственности за доставку данных?
2. Как IP использует поле `Protocol`?
3. Почему ARP невозможен через маршрутизатор?
