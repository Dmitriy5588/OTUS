# VXLAN. Multihoming.
### Цель:
- Настроить отказоустойчивое подключение клиентов с использованием EVPN Multihoming
### Краткое описание
EVPN LAG использует понятие Ethernet segment identifier (ESI). ESI является обязательным атрибутом, который требуется для включения Multihoming.

Одно и то же значение ESI, включенное на нескольких интерфейсах leaf коммутаторах, подключенных к одному серверу или коммутатору, хосту и т.д., формирует EVPN LAG. Этот EVPN LAG поддерживает active-active multihoming. 

Также ESI, LACP System-id должен совпадать на всех устройствах в нашей LAG.

### Схема сети
Общай схема нашего лабораторного стенда
![](https://github.com/Dmitriy5588/OTUS/blob/main/VXLAN.%20Multihoming/Screenshot_1.png)

Для проверки и тестирования Multihoming нам нужен небольшой кусок нашей сети где есть два линка от коммутатора Test_Host в сторону двух разных Leaf1 и Leaf2.

![](https://github.com/Dmitriy5588/OTUS/blob/main/VXLAN.%20Multihoming/Screenshot_1%20%E2%80%94%20%D0%BA%D0%BE%D0%BF%D0%B8%D1%8F.png)

### Настройка сетевых устройств
#### Настройка LAG на Leaf1
```
!
interface Port-Channel100
   switchport trunk allowed vlan 2-4094
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0050:0000:0003:0005:0000
      route-target import 50:00:00:03:00:05
   lacp system-id 5000.0003.0005
!
```
#### Настройка LAG на Leaf2
```
!
interface Port-Channel100
   switchport trunk allowed vlan 2-4094
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0050:0000:0003:0005:0000
      route-target import 50:00:00:03:00:05
   lacp system-id 5000.0003.0005
!
```


