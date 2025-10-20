# Установление соединения

Тема: **Установление соединения (TCP — three-way handshake) и закрытие**

> Кратко: для TCP требуется трехсторонний handshake для согласования начальных порядковых номеров и состояния. UDP — без установления.

### TCP: три шага (упростлено)

1. Клиент → сервер: `SYN`, `Seq = x`.
2. Сервер → клиент: `SYN,ACK`, `Seq = y`, `Ack = x+1`.
3. Клиент → сервер: `ACK`, `Seq = x+1`, `Ack = y+1`.

Формально:

$$
\begin{aligned}
&\text{C}\xrightarrow{SYN,Seq=x}\text{S},\
&\text{S}\xrightarrow{SYN,ACK,Seq=y,;Ack=x+1}\text{C},\
&\text{C}\xrightarrow{ACK,;Seq=x+1,;Ack=y+1}\text{S}.
\end{aligned}
$$

После третьего шага соединение считается установленным (`ESTABLISHED`).

### Параметры, согласуемые при установлении

* Initial Sequence Numbers (ISN).
* Window size (receive window).
* TCP options: MSS, Window Scale, SACK-perm, Timestamps.
* MSS влияет на максимальный полезный payload в сегменте.

### Simultaneous open / passive vs active open

* **Active open** — инициатор вызывает `connect()`.
* **Passive open** — сервер вызывает `listen()` и `accept()`.
* **Simultaneous open** возможен, когда оба хоста отправляют `SYN` одновременно; обмен завершится обычным образом с `SYN,ACK`/`ACK`.

### Закрытие соединения

* Грейсфул: 4-way FIN
  1. A → B: `FIN,Seq = u`
  2. B → A: `ACK, Ack = u+1`
  3. B → A: `FIN, Seq = v`
  4. A → B: `ACK, Ack = v+1`
* TIME\_WAIT: сторона, отправившая последний ACK (обычно клиент), остаётся в `TIME_WAIT` ≈ `2×MSL`. Защищает от старых пакетов и позволяет корректно завершить соединение.
* Быстрый сброс: `RST` обрывает соединение немедленно.

### Проблемы и механизмы защиты

* **SYN flood** — множество незавершённых `SYN` исчерпывают очередь. Защита: SYN cookies, увеличенные очереди, rate-limiting, фильтрация.
* **Handshake delay / retransmits** — если ACK не пришёл, сегменты ретранслируются по RTO; повторные попытки заканчиваются тайм-аутом.
* **Middleboxes** могут менять опции (MSS clamping), влиять на handshake.

### Диагностика (wireshark / tcpdump)

* Найти SYN без ACK: `tcp.flags.syn==1 && tcp.flags.ack==0`.
* Поиск SYN-ACK: `tcp.flags.syn==1 && tcp.flags.ack==1`.
* Показать поток: в Wireshark `tcp.stream eq N`.
* tcpdump: `tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) == 0` (низкоуровневые выражения).

### UDP и «соединение»

* UDP не устанавливает соединение. `connect()` для UDP просто фиксирует удалённого пира в ОС; не производится handshake.

### Краткие экзаменационные тезисы

1. TCP: SYN → SYN/ACK → ACK.
2. ACK подтверждает `Seq+1` инициирующего SYN.
3. TIME\_WAIT = `2×MSL` нужен для защиты от старых пакетов.
4. SYN flood — распространённая атака; SYN cookies — обычная защита.
