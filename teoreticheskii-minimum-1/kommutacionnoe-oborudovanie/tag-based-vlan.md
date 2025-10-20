---
description: назначение, принцип работы, порты access и trunk
---

# Tag based VLAN

> Коротко: VLAN разделяет один физический L2-домен на несколько логических. Тегирование 802.1Q маркирует кадры, чтобы коммутаторы знали, к какой VLAN они принадлежат.

### Что такое VLAN

* VLAN = логическая сеть внутри L2-доменa.
* Цель: сегментация трафика, изоляция broadcast, безопасность и организационная разделённость.

### Диапазон ID

* Стандартный диапазон VLAN ID: $$1..4094$$.
* 0/4095 зарезервированы, 1 часто — default/native.

### 802.1Q — тег Ethernet

* Тег вставляется в Ethernet-кадр между полем EtherType и payload.
* Структура тега: `TPID(0x8100) | TCI (Priority(3) | CFI(1) | VLAN ID(12))`.
* VLAN ID занимает 12 бит.

### Access vs Trunk — поведение портов

* **Access порт**
  * Принимает untagged кадры.
  * При ingress кадр получает PVID (привязанную VLAN) и передаётся дальше с VLAN-id в таблице.
  * Egress — кадры отправляются untagged.
  *   Команда (Cisco):

      ```
      switchport mode access
      switchport access vlan 10
      ```
* **Trunk порт**
  * Переносит кадры нескольких VLAN.
  * На кадрах ставится 802.1Q-тег (tagged) при egress для всех VLAN, кроме native.
  * Поддерживает разрешённый список VLAN.
  *   Команда (Cisco):

      ```
      switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk native 1
      switchport trunk allowed vlan 10,20,30
      ```
* **Native VLAN** — VLAN, чьи кадры при egress отправляются без тега. По умолчанию VLAN 1. Осторожно: вызывает уязвимости (double tagging).

### PVID и поведение при ingress

* PVID = порт-связанный VLAN для untagged кадров.
* Если trunk получает untagged кадр, он присваивает ему PVID (обычно native).

### Коммутаторная логика

* MAC-таблица хранит соответствие `MAC → (VLAN, порт)`.
* Для коммутации учитывается VLAN. Кадр не будет переслан в VLAN, отличный от его тега/PVID.

### Дополнительные возможности

* **Voice VLAN** — отдельный PVID для голосового трафика (обычно динамически через 802.1Q + 802.1p/802.1x).
* **QinQ (802.1ad)** — двойное тегирование для провайдерских сетей.
* **VLAN trunk pruning / allowed list** — ограничивает VLAN на trunk для экономии полосы.

### Примеры Linux (iproute2)

```
# создать субинтерфейс vlan 10 на eth0
ip link add link eth0 name eth0.10 type vlan id 10
ip link set eth0.10 up
ip addr add 192.168.10.1/24 dev eth0.10
```

### Ошибки и уязвимости

* Duplex/auto-neg mismatch не связан с VLAN, но мешает.
* Native VLAN mismatch приводит к loss/loops и security risks (VLAN hopping).
* Оставлять VLAN 1 как management — плохая практика.

### Диагностика (что смотреть)

* Таблица VLAN: `show vlan brief` (Cisco).
* Trunk-статус: `show interfaces trunk`.
* MAC-table по VLAN: `show mac address-table dynamic vlan 10`.
* Захват трафика: в Wireshark смотреть `vlan.id == 10` или `eth.type == 0x8100`.

### Экзаменационные тезисы

1. VLAN логически разделяет L2-домен.
2. 802.1Q добавляет 4-байтовый тег с TPID=0x8100 и 12-бит VLAN ID.
3. Access = untagged in/out. Trunk = tagged for multiple VLANs; native может быть untagged.
4. PVID применяется к untagged кадрам на ingress.
5. QinQ — double tagging для S-VLAN/C-VLAN.

### Вопросы для самопроверки

1. Чем отличается PVID от native VLAN?
2. Почему mismatch native VLAN опасен?
3. Как настроить trunk, чтобы пропускать VLAN 10 и 20 и сделать 99 native?
