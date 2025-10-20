---
description: назначение, общий принцип работы
---

# Протокол STP\RSTP

> Кратко: предотвращают петли в L2-сети. STP строит ацикличное дерево, RSTP делает это быстро.

### Задача

* Устранить L2-петли и broadcast-storm.
* Сохранить резервные пути, блокируя некоторые интерфейсы.

### Основные элементы

* **BPDU** — кадр протокола (Bridge Protocol Data Unit). Передаёт приоритеты, cost, ID мостов.
* **Root Bridge** — мост с наименьшим Bridge ID (priority + MAC).
* **Path Cost** — стоимость пути; суммируется по интерфейсам. Выбирается путь с минимальной суммарной стоимостью.
* **Port Roles**: Root Port (RP), Designated Port (DP), Non-Designated/Blocked.
* **Port States (802.1D)**: Disabled, Blocking, Listening, Learning, Forwarding.
* **Timers (STP)**: hello, max\_age, forward\_delay.

### Алгоритм (упрощённо)

1. Все коммутаторы обмениваются BPDU.
2. Выбирается Root Bridge (наименьший Bridge ID).
3. Для каждого сегмента выбирается Designated Port (порт с лучшим корневым путём).
4. На остальных портах ставится блокировка, чтобы убрать петли.
5. При изменениях пересчёт выполняется по алгоритму выборов.

### Path cost — как считается

* Path cost зависит от пропускной способности интерфейса. В классике используются таблицы (например, для 100Mbps cost=19, 1Gbps cost=4).
* В стандартах возможна формула приближения, но важно знать суть: меньшая стоимость = предпочтительный путь.

### RSTP (802.1w) — отличия и преимущества

* Быстрая конвергенция (Rapid Spanning Tree).
* Упрощённые состояния: Discarding, Learning, Forwarding.
* Negotiation/handshake для быстро перехода в Forwarding на point-to-point линках.
* Edge ports (аналог portfast) — мгновенный переход в Forwarding для хост-портов.
* Backward compatible с классическим STP через BPDU формат.

### Практические механики и настройки

* `spanning-tree vlan <id> root primary/secondary` — настройка приоритета root.
* PortFast / edge — для портов с конечными хостами. Включать только на портах, где нет коммутаторов.
* BPDU Guard — отключает порт при приходе BPDU на PortFast.
* Root Guard — предотвращает, чтобы другой коммутатор стал root.
* UplinkFast/BackboneFast — устаревшие Cisco-опции для ускорения конвергенции в STP.

### Проблемы и рекомендации

* Неправильная настройка PortFast приводит к петлям.
* Native VLAN mismatch + STP может дать неожиданные блокировки.
* Для резервирования в production используют RSTP или Rapid PVST/PVST+.
* Контролируйте таймеры только при полном понимании сети.

### Диагностика (что смотреть)

* `show spanning-tree` — статус root, роли портов, blocked ports.
* BPDU приходят? `show spanning-tree interface <if>`; смотрите BPDU counters.
* Всплески topology change — `topology change` события и пересчёты.

### Экзаменационные тезисы

1. STP строит дерево путём обмена BPDU и блокирует избыток портов.
2. Root Bridge = минимальный Bridge ID.
3. Port roles: Root, Designated, Blocked.
4. RSTP ускоряет конвергенцию и вводит edge ports.
5. PortFast = быстрый переход для хост-портов; использовать осторожно с BPDU Guard.

### Вопросы для самопроверки

1. Как выбирается Root Bridge?
2. Какие роли портов существуют и какова их логика?
3. Чем RSTP принципиально быстрее классического STP?
