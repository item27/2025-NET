# MAC адрес, структура, назначение

> Кратко: MAC — физический идентификатор сетевого интерфейса на канальном уровне. Он используется для локальной доставки кадров в L2-домене.

### Что такое MAC

* MAC (Media Access Control) — уникальный 48-битный идентификатор интерфейса.
* Записывается как шесть байт в шестнадцатеричном виде: `00:11:22:33:44:55`.
* Используется в Ethernet для адресации источника и приёмника в кадре.

### Структура и биты смысла

* 48 бит делятся на OUI (Organizationally Unique Identifier) 24 бита и NIC-specific 24 бита.
* Первый октет содержит служебные биты:
  * младший бит $$b_0$$ — I/G (Individual/Group). $$b_0=1$$ → multicast/broadcast.
  * следующий бит $$b_1$$ — U/L (Universal/Local). $$b_1=0$$ → адрес назначен производителем; $$b_1=1$$ → локально назначен.
* Пример: `01:00:5e:...` — начало IPv4-multicast MAC. `33:33:...` — IPv6-multicast MAC. `ff:ff:ff:ff:ff:ff` — broadcast.

### Типы MAC

* **Unicast** — уникальный адрес одного интерфейса (обычно настоящий NIC).
* **Multicast** — адрес группы (мэппинг из IP-multicast).
* **Broadcast** — все единицы, доставляется всем в L2-домене.
* **Locally administered** — назначен администратором/системой (U/L=1).

### Как MAC используется в сети

* Коммутатор обучает MAC-таблицу по source MAC входящих кадров: `MAC → порт`.
* По destination MAC коммутатор решает, куда отправлять кадр.
* Если MAC неизвестен — флудинг в VLAN.
* Можно настроить статические записи и port-security (жёсткая привязка MAC→порт).

### Связь с ARP и IP

* IP-пакет на L3 инкапсулируется в Ethernet-кадр с MAC-адресами.
* Для сопоставления IP→MAC применяется ARP (IPv4) или NDP/NEIGH (IPv6).

### Где смотреть/команды

* Linux: `ip link show`, `ip neigh`, `ip -s link`
* Старое: `ifconfig -a`
* ARP: `arp -a`
* Switch (Cisco): `show mac address-table`, `show arp`
* Wireshark/tcpdump: `eth.addr == 00:11:22:33:44:55`, `eth.dst == ff:ff:ff:ff:ff:ff`

### Multicast mapping (коротко)

* IPv4 → MAC: `01:00:5e:xx:xx:xx` (нижние 23 бита IP).
* IPv6 → MAC: `33:33:xx:xx:xx:xx` (нижние 32 бита IPv6).

### Проблемы и уязвимости

* MAC-spoofing.
* MAC-flooding (заполнение таблицы) → CAM overflow → флудинг.
* Native/local MAC mismatch при настройке порт-security.

### Экзаменационные тезисы

1. MAC — 48 бит. Формат `HH:HH:HH:HH:HH:HH`.
2. $$b_0$$ (LSB первого октета) = 1 → multicast/group. $$b_1$$ = 1 → локально задан.
3. Switch учит MAC по source и использует таблицу для forwarding.
4. Статические MAC и port-security ограничивают доступ.

### Вопросы для самопроверки

1. Как отличить unicast MAC от multicast по первым байтам?
2. Что произойдёт на коммутаторе, если destination MAC отсутствует в таблице?
3. Как проверить MAC-адрес интерфейса в Linux?
