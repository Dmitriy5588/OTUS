# Построение сети Underlay на основе протокола OSPF.
### Цель:
- Настроить OSPF для Underlay сети;
- Проверить IP связность между сетевыми устройствами.

### Введение
Кратко об __Underlay и OSPF...__

__Underlay__ - это физическая сеть: аппаратные коммутаторы и кабели. Устройства в андерлее знают, как добраться до физических машин.

__OSPF (Open Shortest Path First)__ — протокол динамической маршрутизации, основанный на технологии отслеживания состояния канала и использующий для нахождения кратчайшего пути алгоритм Дейкстры.

__Принцип работы OSPF:__ после включения маршрутизаторов протокол ищет непосредственно подключённых соседей и устанавливает с ними «дружеские» отношения. Затем они обмениваются друг с другом информацией о подключённых и доступных им сетях. То есть они строят карту сети (топологию сети). На основе полученной информации запускается алгоритм SPF (Shortest Path First, «выбор наилучшего пути»), который рассчитывает оптимальный маршрут к каждой сети.

OSPF делит сети на три основных категории, по методу доставки:
 - __Point-to-point:__ сеть, обьединяющая пару маршрутизаторов. DR/BDR не выбирается.
 - __Broadcast:__ широковещательная сеть с множественным доступом, основной представитель – Ethernet. Выбирается DR/BDR.
 - __Nonbroadcast multiaccess (NBMA):__ сеть (обычно WAN), объединяющая более двух маршрутизаторов, но не поддерживающая широковещательную рассылку.

### Описание сети
Топология сети и адреса сетевых устройств были ранее сконфигурированны на лабораторном стенде.
Все сетевые устройства будут находиться в одной автономной системе, номер AREA или "зона" будет нулевой, или как её называют __Backbone-area.__

Тип нашей сети __Point-to-point.__ Адреса интерфесов сетевых устройств настроены с маской __/31__ для p2p линков.
### Схема сети

![](https://github.com/Dmitriy5588/OTUS/blob/main/Underlay.%20OSPF/OSPF.png)

#### Таблица адресов

|_Device_|_Port_|_IP Address_|_Subnet_ _Mask_|_Link_
|---|---|---|---|---|
Spine1|Eth1|10.2.1.0|255.255.255.254|p2p
Spine1|Eth2|10.2.1.2|255.255.255.254|p2p
Spine1|Eth3|10.2.1.4|255.255.255.254|p2p
Spine1|*__Lo1__*|10.0.1.0|255.255.255.255|Loopback1
Spine2|Eth1|10.2.2.0|255.255.255.254|p2p
Spine2|Eth2|10.2.2.2|255.255.255.254|p2p
Spine2|Eth3|10.2.2.4|255.255.255.254|p2p
Spine2|*__Lo1__*|10.0.2.0|255.255.255.255|Loopback1
Leaf1|Eth1|10.2.1.1|255.255.255.254|p2p
Leaf1|Eth2|10.2.2.1|255.255.255.254|p2p
Leaf1|*__Lo1__*|10.0.0.1|255.255.255.255|Loopback1
Leaf2|Eth1|10.2.1.3|255.255.255.254|p2p
Leaf2|Eth2|10.2.2.3|255.255.255.254|p2p
Leaf2|*__Lo1__*|10.0.0.2|255.255.255.255|Loopback1
Leaf3|Eth1|10.2.1.5|255.255.255.254|p2p
Leaf3|Eth2|10.2.2.5|255.255.255.254|p2p
Leaf3|*__Lo1__*|10.0.0.3|255.255.255.255|Loopback1

### Настройка сетевых устройств
#### Настройка OSPF на Spine1
Запускаем процесс OSPF 1, в качестве router-id указываем адрес Loopback1, переводит все интерфейсы в пассивный режим, за исключением наших __p2p__ линков.
```
!
router ospf 1
   router-id 10.0.1.0
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
!
end
Spine1#
```
На интерфейсах указываем тип сети и номер __area__

```
interface Ethernet1
   description to_Leaf1
   no switchport
   ip address 10.2.1.0/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description to_Leaf2
   no switchport
   ip address 10.2.1.2/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description to_Leaf3
   no switchport
   ip address 10.2.1.4/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
```

*__Аналогично выполняем настройки на всех сетевых устройствах в нашей AREA.__*
#### Настройка OSPF на Spine2
```
!
interface Ethernet1
   description to_Leaf1
   no switchport
   ip address 10.2.2.0/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description to_Leaf2
   no switchport
   ip address 10.2.2.2/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description to_Leaf3
   no switchport
   ip address 10.2.2.4/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 10.0.2.0/32
   ip ospf area 0.0.0.0
!
ip routing
!
router ospf 1
   router-id 10.0.2.0
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
!
end
Spine2#
```
#### Настройка OSPF на Leaf1
```
!
interface Ethernet1
   description to_Spine1
   no switchport
   ip address 10.2.1.1/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description to_Spine2
   no switchport
   ip address 10.2.2.1/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 10.0.0.1/32
   ip ospf area 0.0.0.0
!
ip routing
!
router ospf 1
   router-id 10.0.0.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
end
Leaf1#
```
#### Настройка OSPF на Leaf2
```
interface Ethernet1
   description to_Spine1
   no switchport
   ip address 10.2.1.3/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description to_Spine2
   no switchport
   ip address 10.2.2.3/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 10.0.0.2/32
   ip ospf area 0.0.0.0
!
ip routing
!
router ospf 1
   router-id 10.0.0.2
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
end
Leaf2#
```
#### Настройка OSPF на Leaf3
```
interface Ethernet1
   description to_Spine1
   no switchport
   ip address 10.2.1.5/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description to_Spine2
   no switchport
   ip address 10.2.2.5/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 10.0.0.3/32
   ip ospf area 0.0.0.0
!
ip routing
!
router ospf 1
   router-id 10.0.0.3
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
end
Leaf3#
```
### Проверка установления соседства с устройствами в сети
Выполним команду __*show ip ospf neighbor*__ на устройстве Spine1.
```
Spine1#show ip ospf neighbor
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.0.0.1        1        default  0   FULL                   00:00:30    10.2.1.1        Ethernet1
10.0.0.2        1        default  0   FULL                   00:00:34    10.2.1.3        Ethernet2
10.0.0.3        1        default  0   FULL                   00:00:36    10.2.1.5        Ethernet3
Spine1#
```
Вывод команды отображает все отношения соседства.

### Проверка IP связности устройств и заполнение таблицы маршрутизации
Для проверки выполним команду __*ping*__ c устройства Spine1 до Spine2. В качестве source укажем loopback1:
```
Spine1#ping 10.0.2.0 source loopback 1
PING 10.0.2.0 (10.0.2.0) from 10.0.1.0 : 72(100) bytes of data.
80 bytes from 10.0.2.0: icmp_seq=1 ttl=63 time=12.3 ms
80 bytes from 10.0.2.0: icmp_seq=2 ttl=63 time=4.79 ms
80 bytes from 10.0.2.0: icmp_seq=3 ttl=63 time=5.05 ms
80 bytes from 10.0.2.0: icmp_seq=4 ttl=63 time=6.35 ms
80 bytes from 10.0.2.0: icmp_seq=5 ttl=63 time=4.62 ms

--- 10.0.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 45ms
rtt min/avg/max/mdev = 4.627/6.632/12.340/2.918 ms, ipg/ewma 11.326/9.391 ms
Spine1#
```
IP связность с устройством Spine2 есть.

Для проверки таблицы маршрутизации OSPF выполним команду __*show ip route ospf*.__
```
Spine1#show ip route ospf

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

 O        10.0.0.1/32 [110/20] via 10.2.1.1, Ethernet1
 O        10.0.0.2/32 [110/20] via 10.2.1.3, Ethernet2
 O        10.0.0.3/32 [110/20] via 10.2.1.5, Ethernet3
 O        10.0.2.0/32 [110/30] via 10.2.1.1, Ethernet1
                               via 10.2.1.3, Ethernet2
                               via 10.2.1.5, Ethernet3
 O        10.2.2.0/31 [110/20] via 10.2.1.1, Ethernet1
 O        10.2.2.2/31 [110/20] via 10.2.1.3, Ethernet2
 O        10.2.2.4/31 [110/20] via 10.2.1.5, Ethernet3

Spine1#
```
Как видно из вывода таблицы маршрутизации все подсети успешно изучены.

Дополнительно выборочно проверим связность других устройств.

IP связность между Leaf1 и Leaf3:
```
Leaf1#ping 10.0.0.3 source loopback 1
PING 10.0.0.3 (10.0.0.3) from 10.0.0.1 : 72(100) bytes of data.
80 bytes from 10.0.0.3: icmp_seq=1 ttl=63 time=8.75 ms
80 bytes from 10.0.0.3: icmp_seq=2 ttl=63 time=4.25 ms
80 bytes from 10.0.0.3: icmp_seq=3 ttl=63 time=5.43 ms
80 bytes from 10.0.0.3: icmp_seq=4 ttl=63 time=4.15 ms
80 bytes from 10.0.0.3: icmp_seq=5 ttl=63 time=4.03 ms

--- 10.0.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 32ms
rtt min/avg/max/mdev = 4.039/5.327/8.756/1.787 ms, ipg/ewma 8.166/6.968 ms
Leaf1#
```
IP связность между Leaf2 и Leaf3.
```
Leaf2#ping 10.0.0.3 source loopback 1
PING 10.0.0.3 (10.0.0.3) from 10.0.0.2 : 72(100) bytes of data.
80 bytes from 10.0.0.3: icmp_seq=1 ttl=63 time=8.31 ms
80 bytes from 10.0.0.3: icmp_seq=2 ttl=63 time=4.70 ms
80 bytes from 10.0.0.3: icmp_seq=3 ttl=63 time=5.33 ms
80 bytes from 10.0.0.3: icmp_seq=4 ttl=63 time=4.27 ms
80 bytes from 10.0.0.3: icmp_seq=5 ttl=63 time=3.91 ms

--- 10.0.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 32ms
rtt min/avg/max/mdev = 3.912/5.307/8.315/1.577 ms, ipg/ewma 8.027/6.734 ms
Leaf2#
```
Также таблица маршрутизации на Leaf3.
```
VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

 O        10.0.0.1/32 [110/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 O        10.0.0.2/32 [110/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 O        10.0.1.0/32 [110/20] via 10.2.1.4, Ethernet1
 O        10.0.2.0/32 [110/20] via 10.2.2.4, Ethernet2
 O        10.2.1.0/31 [110/20] via 10.2.1.4, Ethernet1
 O        10.2.1.2/31 [110/20] via 10.2.1.4, Ethernet1
 O        10.2.2.0/31 [110/20] via 10.2.2.4, Ethernet2
 O        10.2.2.2/31 [110/20] via 10.2.2.4, Ethernet2

Leaf3#
```
