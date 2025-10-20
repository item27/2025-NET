# Режим работы (duplex)

> Кратко: duplex определяет, можно ли передавать и принимать одновременно. Full — двунаправленно одновременно. Half — однонаправленно, требуется механизм доступа (CSMA/CD).

### Что такое

* **Full duplex** — передача и приём одновременно на одном канале. Нет коллизий.
* **Half duplex** — передача или приём в момент времени. Возможны коллизии. Требуется CSMA/CD для Ethernet в half-duplex.

### Почему важно

* Неправильный режим приводит к падению производительности.
* **Duplex mismatch** (одна сторона full, другая half) вызывает потерю пакетов, высокая латентность и ошибки.

### Автонеготиация (auto-negotiation)

* Процесс согласования скорости и duplex между NIC и портом коммутатора.
* Рекомендуется включать autoneg на обеих сторонах.
* Для 1000BASE-T autoneg обязательна по стандарту.

### Симптомы duplex mismatch

* Частые коллизии и late collisions.
* Высокий процент retransmits (TCP), низкая полезная скорость.
* На интерфейсе счётчики `collisions`, `tx errors`, `rx errors`, `frame`, `late collision`.

### Диагностика (команды)

* Linux:
  * `ethtool eth0` → строка `Duplex: Full` / `Half` и `Speed`.
  * `cat /sys/class/net/eth0/carrier` и `ip -s link show eth0`.
  * `ifconfig eth0` или `ip -s link` для ошибок/коллизий.
* Cisco:
  * `show interface GigabitEthernet0/1` → строки `duplex`, `speed`, `input errors`, `late collisions`.
* Windows (PowerShell):
  * `Get-NetAdapterAutoconfiguration` / `Get-NetAdapter | Format-List Name, Status, LinkSpeed`.

### Что смотреть в логике

1. Сравнить настройки на обеих сторонах (speed/duplex/autoneg).
2. Если одна сторона fixed и другая autoneg → возможен mismatch.
3. При mismatch либо включить autoneg с обеих сторон, либо вручную задать одинаковые параметры.

### Практические рекомендации

* Включайте autoneg на всех портах, если оборудование поддерживает.
* Для 1Gbps и выше полагайтесь на autoneg; фиксированную настройку используйте только по причине совместимости и после теста.
* При проблемах временно поставьте оба конца в одинаковый фиксированный режим и проверьте поведение.
* Проверяйте качество кабеля и парность при странных ошибках — физическая причина может маскироваться под duplex issue.

### Экзаменационные тезисы

1. Full duplex = simultaneous TX/RX, no collisions.
2. Half duplex = shared medium, CSMA/CD, возможны коллизии.
3. Autoneg согласует speed + duplex; mismatch — частая причина проблем.
4. Диагностика через `ethtool` / `show interface` и счётчики ошибок.

### Вопросы для самопроверки

1. Какие симптомы у duplex mismatch?
2. Почему на 1Gbps autoneg обязателен?
3. Как быстро проверить текущий режим duplex в Linux?
