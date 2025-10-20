---
description: в Wireshark и tcpdump
---

# Простые фильтры по адресам и портам

## Ключевая разница

* **tcpdump / capture filter (BPF)** — фильтр применяется при захвате. Синтаксис: `host`, `net`, `port`, `tcp`, `udp`, `and/or/not`, скобки.
* **Wireshark / display filter** — фильтр применяется к уже захваченным пакетам (более мощный и точный). Синтаксис: `field == value`, `&&`, `||`, `!`.

***

### tcpdump (BPF) — примеры

```bash
# Захват всего трафика к/от хоста
tcpdump -i eth0 'host 192.0.2.10' -w out.pcap

# Только исходящий трафик от хоста
tcpdump -i eth0 'src host 192.0.2.10'

# Только приходящий на порт 80
tcpdump -i eth0 'dst port 80'

# Любые TCP-пакеты на порт 443 (HTTPS)
tcpdump -i eth0 'tcp and port 443'

# UDP DNS (порт 53)
tcpdump -i eth0 'udp and port 53'

# Сеть 192.168.0.0/16
tcpdump -i eth0 'net 192.168.0.0/16'

# Диапазон портов 8000-8100
tcpdump -i eth0 'portrange 8000-8100'

# Комбинации (скобки важны)
tcpdump -i eth0 '(src net 10.0.0.0/8 and tcp) and (dst port 22 or dst port 2222)'

# MAC-адрес
tcpdump -i eth0 'ether host 00:11:22:33:44:55'

# Multicast или broadcast
tcpdump -i eth0 'multicast' 
tcpdump -i eth0 'broadcast'

# Rекомендации: не разрешать DNS-имена (быстрее и читабельнее)
tcpdump -i eth0 -nn -s 0 'tcp and port 80'
```

***

### Wireshark — display filters (примеры)

```
# Любой пакет с IP-адресом 192.0.2.10 (src или dst)
ip.addr == 192.0.2.10

# Только источником 192.0.2.10
ip.src == 192.0.2.10

# Только назначению 10.0.0.5
ip.dst == 10.0.0.5

# TCP-порт (любой: src или dst)
tcp.port == 443

# Конкретно dst-port TCP
tcp.dstport == 443
tcp.srcport == 12345

# UDP-порт
udp.port == 53

# MAC-адрес
eth.addr == 00:11:22:33:44:55

# IPv6
ipv6.addr == 2001:db8::1
ipv6.src == fe80::1

# SYN без ACK (новые соединения)
tcp.flags.syn == 1 && tcp.flags.ack == 0

# ICMPv6 Neighbor Solicitation
icmpv6.type == 135

# Логические комбинации
ip.src == 10.0.0.1 && tcp.dstport == 80
(ip.addr == 192.0.2.10 || ip.addr == 192.0.2.11) && udp.port == 53
```

***

### Быстрые готовые сценарии

* Захват HTTPS и просмотр в Wireshark:

```bash
tcpdump -i eth0 -nn -s 0 'tcp and port 443' -w https.pcap
# затем в Wireshark: tcp.port == 443 && ip.addr == <host>
```

* Если ICMP/UDP блокированы, тестировать TCP-путь (Wireshark display):

```bash
# Захват TCP SYN/SYN-ACK интерактивно
tcpdump -i eth0 -nn 'tcp[tcpflags] & (tcp-syn) != 0' -w syns.pcap
# Или в Wireshark: tcp.flags.syn == 1
```

***

### Советы и подводные камни

* **BPF (tcpdump) не понимает поля Wireshark**. BPF ограничен ключевыми словами и порт/host/net.
* **Всегда ставьте кавычки** вокруг сложных выражений в командной строки.
* Для tcpdump используйте `-nn` (без разрешения имён) и `-s 0` (полная длина пакета).
* Для IPv6 в BPF используйте `ip6` ключевые слова: `tcpdump -i eth0 'ip6 and host 2001:db8::1'`.
* Если capture-filter слишком жёсткий, потеряете пакеты; если display-filter — сильнее гибкость для анализа.

