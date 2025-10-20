# Port Mirroring

> Кратко: зеркалирование дублирует трафик с одного/нескольких портов или VLAN на целевой порт/сервер для анализа или записи. Используется для мониторинга, IDS/IPS, отладки и аудита.

### Основные понятия

* **Source** — порт/VLAN, трафик которого копируют.
* **Destination** — порт, на который отправляют копии (анализатор/Wireshark/IDS).
* **Direction** — `ingress`, `egress` или `both`.
* **Local SPAN** — зеркалирование внутри одного коммутатора.
* **RSPAN (Remote SPAN)** — копирование в специальный VLAN, который транпортирует зеркалируемый трафик к другому коммутатору.
* **ERSPAN** — зеркалирование через туннель (GRE) между коммутаторами, позволяет отправлять зеркал по L3.

### Когда использовать

* Снимки трафика для расследования инцидентов.
* Непрерывный мониторинг через IDS/NGFW.
* Анализ потоков с проблемами (потери, задержки, retransmits).

### Ограничения и риски

* Целевая линия/порт может стать бутылочным горлышком. Пакеты могут теряться при высокой нагрузке.
* Возможна модификация порядка/меток VLAN при RSPAN/ERSPAN.
* На аппаратных SPAN портах возможна отсечка длинных пакетов или отсутствие репликации некоторых типов мета-пакетов (напм. control-plane).
* Юридические и конфиденциальные ограничения при захвате трафика.

### Примеры конфигурации (коротко)

#### Cisco — локальный SPAN

```bash
monitor session 1 source interface GigabitEthernet0/1 both
monitor session 1 destination interface GigabitEthernet0/10
```

* Можно указать `source vlan 10` или `source interface Gi0/1,Gi0/2`.
* `both` = ingress+egress.

#### Cisco — RSPAN (упрощённо)

```bash
vlan 100
  remote-span
!
monitor session 1 source interface Gi0/1 both
monitor session 1 destination remote vlan 100
```

* На удалённом коммутаторе VLAN100 привязан к physical порту для анализа.

#### Cisco — ERSPAN (GRE) (примерная схема)

* Конфигурация ERSPAN требует указания source/dest IP и типа ERSPAN; на разных платформах команды различаются. ERSPAN инкапсулирует mirrored frame в GRE и отправляет по IP.

#### Linux — `tc mirred` (пример)

```bash
# включаем ingress qdisc
tc qdisc add dev eth0 ingress

# зеркалим всё ingress eth0 на eth1 (destination)
tc filter add dev eth0 parent ffff: protocol ip u32 match u32 0 0 action mirred egress mirror dev eth1
```

* `mirred` доступен в современных ядрах. Для сложных фильтров используют `u32`, `bpf` и т.д.

#### Linux — iptables TEE (копирование на IP-адрес анализатора)

```bash
# дублирует пакеты на Collector_IP (нужен модуль TEE)
iptables -t mangle -A PREROUTING -j TEE --gateway 192.0.2.10
```

* Копия инкапсулируется в IP. Подходит когда нужно переслать копию на удалённый анализатор.

### Практические советы

* Направляйте зеркалируемый поток на выделенный анализатор с высокой пропускной способностью и быстрым диском/сетью.
* Используйте фильтры SPAN/`tc` чтобы не перегружать канал (фильтруйте по VLAN/port/IP/port range).
* Для long-term capture применяйте sampling или агрегирование (NetFlow/sFlow) вместо полного зеркалирования.
* Проверяйте `show monitor session` (Cisco) или `tc -s filter show` (Linux) для статистики потерь.
* На целевом интерфейсе включите promiscuous mode и используйте `tcpdump -i ethX -w file.pcap` или Wireshark.

### Диагностика ошибок

* На коммутаторе: `show monitor session all` / `show interfaces counters` / CPU spike при зеркалировании.
* Если нет пакетов в анализаторе — проверить направление (ingress/egress), VLAN native/tagging, MTU (ERSPAN увеличивает заголовок).
* При RSPAN/ERSPAN проверьте, что транзитные точки не фильтруют VLAN/GRE.

### Краткие тезисы для экзамена

1. SPAN = локальное зеркалирование; RSPAN = через VLAN; ERSPAN = через L3 (GRE).
2. Направление: ingress, egress, both.
3. Ограничение: целевой порт может терять пакеты при высокой нагрузке.
4. На Linux можно использовать `tc mirred` или `iptables TEE` для дублирования.

### Вопросы для самопроверки

1. В чём разница между RSPAN и ERSPAN?
2. Почему нельзя зеркалить трафик 1G на целевой порт 100Mb без потерь?
3. Как сократить нагрузку на зеркало при анализе только HTTP-трафика?
