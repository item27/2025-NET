# Отказ от бродкста (IPv6)

> кратко: в IPv6 нет L2/L3 broadcast. Вместо него используются multicast и NDP (Neighbor Discovery). Это уменьшает шум сети и повышает масштабируемость.

### Что изменилось

* IPv6 убирает broadcast.
* Широковещательные функции IPv4 заменены multicast и ICMPv6-пакетами.
* ARP заменён на NDP через ICMPv6.

### Основные механизмы вместо broadcast

* **All-nodes multicast** `ff02::1` для сообщений всем узлам на link.
*   **Solicited-node multicast** для разрешения соседей. Адрес формируется как базовый `ff02::1:ff00:0` плюс младшие 24 бита IPv6-адреса:

    $$
    \text{Solicited} = \text{ff02::1:ff00:0} + (\text{IPv6}_{\text{low-24}})
    $$
* **NDP** (ICMPv6 types 135/136) выполняет функции ARP, Router Discovery, Prefix Discovery, Address Autoconf.

### Почему это лучше

* Уменьшение broadcast-storms.
* Таргетированная доставка при разрешении соседей.
* Меньше лишней нагрузки на все хосты в сегменте.

### Что важно для сети

* Коммутаторы должны поддерживать IPv6 multicast snooping/MLD snooping, чтобы ограничивать flooding.
* Протоколы управления соседями требуют корректной поддержки ICMPv6 на устройствах и маршрутизаторах.
* Многие сервисы и утилиты используют multicast, не broadcast. Нужна проверка firewall правил для ICMPv6 и multicast.

### Безопасность и риски

* Уязвимости: Rogue RA (Router Advertisement), NDP-spoofing.
* Меры защиты: RA Guard, DHCPv6 Guard, SEND (Secure Neighbor Discovery), фильтрация multicast на границе.
* Логи и мониторинг ICMPv6 важны для обнаружения атак.

### Практика и диагностика

* Захват ICMPv6:

```bash
tcpdump -n -i eth0 'icmp6'
```

* Отслеживать NDP:

```bash
ip -6 neigh show
```

* Фильтры в Wireshark: `icmpv6.type == 135` (Neighbor Solicitation) и `icmpv6.type == 136` (Neighbor Advertisement).
* Проверить мультикаст-подписки: `ip -6 maddr show`.

### Экзаменационные тезисы

1. IPv6 не использует broadcast.
2. ARP заменён на NDP через ICMPv6.
3. Solicited-node multicast содержит последние 24 бита адреса.
4. Коммутаторы должны заботиться о MLD snooping, чтобы избежать флуда.
5. Защищать сеть от rogue RA и NDP-spoofing.

### Вопросы для самопроверки

1. Как формируется solicited-node multicast адрес для `2001:db8::1a2b:3cff:fe4d:5e6f`?
2. Чем NDP принципиально отличается от ARP?
3. Какие защитные механизмы применяют против rogue RA?
