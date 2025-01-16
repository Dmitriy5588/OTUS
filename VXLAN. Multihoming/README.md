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

Для проверки и тестирования Multihoming нам нужно два линка от коммутатора Test_Host в сторону двух Leaf1 и Leaf2.

![](https://github.com/Dmitriy5588/OTUS/blob/main/VXLAN.%20Multihoming/Screenshot_1%20%E2%80%94%20%D0%BA%D0%BE%D0%BF%D0%B8%D1%8F.png)

### Настройка сетевых устройств
Добавим в конфигурацию интерфейс Port-Channel 100. В настройках ESI вручную укажем id и lacp system-id, преобразовав из mac адреса интерфейса port-channel

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
#### Настройка LACP на коммутаторе Test_Host
```
!
interface Port-Channel100
   switchport trunk allowed vlan 2-4094
   switchport mode trunk
!
interface Ethernet1
   channel-group 100 mode active
!
interface Ethernet2
   channel-group 100 mode active
!
```

### Проверка ESI LAG и связности между устройствами
Выполним show команды на устройстве Test_Host:
```
Test_Host#show lacp peer
State: A = Active, P = Passive; S=ShortTimeout, L=LongTimeout;
       G = Aggregable, I = Individual; s+=InSync, s-=OutOfSync;
       C = Collecting, X = state machine expired,
       D = Distributing, d = default neighbor state
                 |                        Partner
 Port    Status  | Sys-id                    Port#   State     OperKey  PortPri
------ ----------|------------------------- ------- --------- --------- -------
Port Channel Port-Channel100:
 Et1     Bundled | 8000,50-00-00-03-00-05        5   ALGs+CD    0x0064    32768
 Et2     Bundled | 8000,50-00-00-03-00-05        5   ALGs+CD    0x0064    32768
```
```
Test_Host#show port-channel 100 detailed
Port Channel Port-Channel100 (Fallback State: Unconfigured):
Minimum links: unconfigured
Minimum speed: unconfigured
Current weight/Max weight: 2/16
  Active Ports:
       Port            Time Became Active       Protocol       Mode      Weight
    --------------- ------------------------ -------------- ------------ ------
       Ethernet1       11:35:59                 LACP           Active      1
       Ethernet2       11:36:17                 LACP           Active      1

Test_Host#
```

Как видно из вывода агрегированный канал собран с нашим system-id.

Для тестирования отказоувстойчивости запустим команду __ping__ до устройства находящимся за другим Leaf2 коммутатором и в процессе отключим один линк из агрегированного канал.

VPC1 - 192.168.2.30

VPC2 - 192.168.2.100

```
VPCS> ping 192.168.2.100 -c 100

84 bytes from 192.168.2.100 icmp_seq=1 ttl=64 time=22.300 ms
84 bytes from 192.168.2.100 icmp_seq=2 ttl=64 time=13.058 ms
84 bytes from 192.168.2.100 icmp_seq=3 ttl=64 time=15.373 ms
84 bytes from 192.168.2.100 icmp_seq=4 ttl=64 time=15.990 ms
84 bytes from 192.168.2.100 icmp_seq=5 ttl=64 time=11.844 ms
84 bytes from 192.168.2.100 icmp_seq=6 ttl=64 time=35.531 ms
84 bytes from 192.168.2.100 icmp_seq=7 ttl=64 time=22.576 ms
84 bytes from 192.168.2.100 icmp_seq=8 ttl=64 time=22.621 ms
84 bytes from 192.168.2.100 icmp_seq=9 ttl=64 time=15.248 ms
84 bytes from 192.168.2.100 icmp_seq=10 ttl=64 time=11.657 ms
84 bytes from 192.168.2.100 icmp_seq=11 ttl=64 time=12.471 ms
```
```
Leaf1(config)#interface Ethernet 5
Leaf1(config-if-Et5)#shutdown
Leaf1(config-if-Et5)#
```

Выполнение этой команды не прекращает ICMP запросы до устройства.




