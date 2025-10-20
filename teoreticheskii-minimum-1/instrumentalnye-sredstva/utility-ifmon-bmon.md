---
description: что они позволяют «увидеть»?
---

# Утилиты ifmon, bmon

## Кратко

* `ifmon` — не единый стандартный инструмент на Linux. Под названием `ifmon` бывают продукт-специфичные решения (например SAP/Interface Monitor) или маленькие скрипты/утилиты в отдельных сборках; в общей практике вместо него используют `bmon`, `iftop`, `ifstat`, `nload` и т.д. ([userapps.support.sap.com](https://userapps.support.sap.com/sap/support/knowledge/en/3212324?utm_source=chatgpt.com))
* `bmon` — лёгкий curses-инструмент для реального времени: собирает статистику по интерфейсам, показывает графы RX/TX, пакеты, ошибки и подробные метрики; интерактивен и скриптуем. ([linux.die.net](https://linux.die.net/man/1/bmon?utm_source=chatgpt.com))

***

## ifmon — что это и что делать, если его нет

* Под «ifmon» часто понимают generic «interface monitor». Однако нет одного общесистемного пакета с этим именем; в ваших системах под «ifmon» может скрываться SAP-модуль или пользовательский скрипт. Если команда `ifmon` не найдена, используйте стандартные утилиты: `ip -s link`, `ifstat`, `iftop`, `nload`, `bmon` или `vnstat`. ([userapps.support.sap.com](https://userapps.support.sap.com/sap/support/knowledge/en/3212324?utm_source=chatgpt.com))
*   Быстрая проверка без установки:

    ```bash
    ip -s link show eth0     # счётчики bytes/packets/errors
    watch -n1 'ip -s link show eth0'
    ```
* Если хотите именно «если-монитор» в стиле tiny script — обычно делают цикл, читающий `/proc/net/dev` и считающий дельты каждые N секунд (примерная логика; реализацию проще найти/написать локально).

***

## bmon — практическое руководство (установить, запустить, читать)

*   Установка (Debian/Ubuntu):

    ```bash
    sudo apt update
    sudo apt install bmon
    ```

    (доступен в репозиториях большинства дистрибутивов). ([tecmint.com](https://www.tecmint.com/bmon-network-bandwidth-monitoring-debugging-linux/?utm_source=chatgpt.com))
*   Запуск:

    ```bash
    bmon              # интерактивный режим, покажет все интерфейсы
    bmon -p eth0      # сразу открыть конкретный интерфейс
    bmon -o ascii:help  # показать возможные output-модули / справку
    ```

    ([tecmint.com](https://www.tecmint.com/bmon-network-bandwidth-monitoring-debugging-linux/?utm_source=chatgpt.com))
* Клавиши и навигация (интерактивно): `h/j/k/l` или стрелки — переключение интерфейсов; `d` — подробные статистики; `g` — граф; `s` — статистика; `a` — абсолютные/скорости; `p` — проценты; `?` — help; `q` — выход. ([Linux Command Library](https://linuxcommandlibrary.com/man/bmon?utm_source=chatgpt.com))
* Что показывает: RX/TX скорость (bps), пакеты/средние/макс, ошибки, drop, MAC/MTU и дополнительные counters (в деталях). Поддерживает input-модули (netlink/procfs) и текстовый/script-friendly вывод. ([linux.die.net](https://linux.die.net/man/1/bmon?utm_source=chatgpt.com))

***

## Когда использовать что

* Нужна лёгкая реальная визуализация по интерфейсам → `bmon` или `nload`. ([tecmint.com](https://www.tecmint.com/bmon-network-bandwidth-monitoring-debugging-linux/?utm_source=chatgpt.com))
* Нужно смотреть per-connection bandwidth → `iftop`.
* Нужен короткий поток дельт байтов/пакетов для скриптов → `ifstat` или парсинг `/proc/net/dev`.
* Нужно долгосрочное логирование → `vnstat` (стационарный сбор). ([phoenixNAP | Global IT Services](https://phoenixnap.com/kb/linux-network-bandwidth-monitor-traffic?utm_source=chatgpt.com))

***

## Быстрые команды-заметки (шпаргалка)

```bash
# посмотреть интерфейсы и счётчики
ip -s link show

# bmon интерактивно
sudo bmon
bmon -p enp0s3

# iftop (per-connection)
sudo apt install iftop
sudo iftop -i enp0s3

# ifstat (дельты, для скриптов)
sudo apt install ifstat
ifstat 1  # обновление каждую секунду

# vnstat (история)
sudo apt install vnstat
vnstat -l -i enp0s3   # live
vnstat -m              # месячные статистики
```

***

## Интерпретация данных — что смотреть первым делом

1. RX/TX speed — соответствуют ли ожиданиям нагрузки.
2. Errors / drop / fifo / collisions — симптомы проблем L1/L2.
3. Spike/постоянная нагрузка на одном интерфейсе → проверить процессы/сессии (`ss`, `iftop`, `lsof`).
4. Небольшие пиковые потери только на промежуточных хопах обычно не критичны; если потеря у конечного хоста — проблема end-to-end. (общая практика — сопоставлять данные инструментов). ([phoenixNAP | Global IT Services](https://phoenixnap.com/kb/linux-network-bandwidth-monitor-traffic?utm_source=chatgpt.com))

***

## Короткие экзаменационные тезисы

* `ifmon` — не стандартизован; уточняйте контекст. Используйте `ip -s link`, `ifstat`, `iftop`, `nload`, `bmon` в зависимости от задачи. ([userapps.support.sap.com](https://userapps.support.sap.com/sap/support/knowledge/en/3212324?utm_source=chatgpt.com))
* `bmon` — удобен для быстрого интерактивного просмотра RX/TX, графов и подробных counters; имеет скриптуемый вывод и набор клавиш для навигации. ([linux.die.net](https://linux.die.net/man/1/bmon?utm_source=chatgpt.com))
