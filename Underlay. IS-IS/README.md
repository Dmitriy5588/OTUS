# Построение сети Underlay на основе протокола IS-IS
### Цель:
- Настроить IS-IS для Underlay сети;
- Проверить IP связность между сетевыми устройствами.
### Краткое описание протокола
IS-IS также как и OSPF является протоколом иерархическим, с возможностью разделения топологии на area. Во-первых, маршрутизаторы IS-IS домена целиком и полностью принадлежат какой-то одной area, то есть граница между областями проходит по линку, а не по маршрутизатору как у OSPF. Во-вторых, в IS-IS нет никакого специального номера area (как area 0 в OSPF), области, на которые разбита топология, могут носить совершенно произвольные номера.
Обязательной настройкой в IS-IS является указание NET (Network Entity Title) адреса, который назначается каждому маршрутизатору после запуска IS-IS процесса.
Структура NET на рисунке ниже.

![](https://github.com/Dmitriy5588/OTUS/blob/main/Underlay.%20IS-IS/NET%20%D0%B0%D0%B4%D1%80%D0%B5%D1%81.png)

 - __AFI__ (Authority and Format Identifier) - чаще всего это поле указывают равным 49
 - __Area ID,__ номер AREA к которой принадлежит маршрутизатор
 - __System ID,__ это идентификатор маршрутизатора, аналог router-id в OSPF
 - __Selector,__ значение равное 00, обозначает в NET адресах, что адрес принадлежит самому маршрутизатору

Существует два уровня взаимодействия маршрутизаторов - __Level 1__ и __Level 2__ (L1 и L2 соответственно).
- Соседство уровня 1 (L1) формируется только между маршрутизаторами одной area.
- Соседство уровня 2 (L2) может быть сформировано как между маршрутизаторами одной area, так и между маршрутизаторами разных area.
  
### Описание сети
Топология сети и адреса сетевых устройств были ранее сконфигурированны на лабораторном стенде.
Все сетевые устройства будут находиться в одной AREA.

Тип нашей сети __Point-to-point.__ Адреса интерфесов сетевых устройств настроены с маской __/31__ для p2p линков.
### Схема сети
![](https://github.com/Dmitriy5588/OTUS/blob/main/Underlay.%20IS-IS/IS-IS.png)
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
#### Настройка IS-IS на Spine1
Запускаем процес IS-IS, указываем NET, где:
где 49 - указывает тип адреса, 0100 - номер нашей зоны, ХХХХ.ХХХХ.ХХХХ - ID устройства, в качестве System ID будем преобразовывать ip адрес Loopback1, 00 - селектор (всегда ноль).
```
router isis 100
   net 49.0100.0100.0000.1000.00
   !
   address-family ipv4 unicast
```
Команда __address-family__ включает семейства адресов, которые IS-IS будет маршрутизировать, и переводит коммутатор в режим конфигурации для этого семейства адресов.

__Настройки на интерфейсах__
```
!
interface Ethernet1
   description to_Leaf1
   no switchport
   ip address 10.2.1.0/31
   isis enable 100
   isis network point-to-point
!
interface Ethernet2
   description to_Leaf2
   no switchport
   ip address 10.2.1.2/31
   isis enable 100
   isis network point-to-point
!
interface Ethernet3
   description to_Leaf3
   no switchport
   ip address 10.2.1.4/31
   isis enable 100
   isis network point-to-point
!
interface Loopback1
   ip address 10.0.1.0/32
   isis enable 100
!
```
#### Настройка IS-IS на остальных сетевых устройствах

<details>
  
<summary> Настройки на Spine2 </summary>

```
!
interface Ethernet1
   description to_Leaf1
   no switchport
   ip address 10.2.2.0/31
   isis enable 100
   isis network point-to-point
!
interface Ethernet2
   description to_Leaf2
   no switchport
   ip address 10.2.2.2/31
   isis enable 100
   isis network point-to-point
!
interface Ethernet3
   description to_Leaf3
   no switchport
   ip address 10.2.2.4/31
   isis enable 100
   isis network point-to-point
!
interface Loopback1
   ip address 10.0.2.0/32
   isis enable 100
!
ip routing
!
router isis 100
   net 49.0100.0100.0000.2000.00
   !
   address-family ipv4 unicast
!
end
Spine2#
```
</details>

<details>
  
<summary> Настройки на Leaf1 </summary>

```
!
interface Ethernet1
   description to_Spine1
   no switchport
   ip address 10.2.1.1/31
   isis enable 100
   isis network point-to-point
!
interface Ethernet2
   description to_Spine2
   no switchport
   ip address 10.2.2.1/31
   isis enable 100
   isis network point-to-point
!
interface Loopback1
   ip address 10.0.0.1/32
   isis enable 100
!
ip routing
!
router isis 100
   net 49.0100.0100.0000.0001.00
   !
   address-family ipv4 unicast
!
end
Leaf1#
```
</details>

<details>
  
<summary> Настройки на Leaf2 </summary>

```
!
interface Ethernet1
   description to_Spine1
   no switchport
   ip address 10.2.1.3/31
   isis enable 100
   isis network point-to-point
!
interface Ethernet2
   description to_Spine2
   no switchport
   ip address 10.2.2.3/31
   isis enable 100
   isis network point-to-point
!
interface Loopback1
   ip address 10.0.0.2/32
   isis enable 100
!
ip routing
!
router isis 100
   net 49.0100.0100.0000.0002.00
   !
   address-family ipv4 unicast
!
end
Leaf2#
```
</details>

<details>
  
<summary> Настройки на Leaf3 </summary>

```
!
interface Ethernet1
   description to_Spine1
   no switchport
   ip address 10.2.1.5/31
   isis enable 100
   isis network point-to-point
!
interface Ethernet2
   description to_Spine2
   no switchport
   ip address 10.2.2.5/31
   isis enable 100
   isis network point-to-point
!
interface Loopback1
   ip address 10.0.0.3/32
   isis enable 100
!
ip routing
!
router isis 100
   net 49.0100.0100.0000.0003.00
   !
   address-family ipv4 unicast
!
end
Leaf3#
```
</details>

### Проверка установления соседства с устройствами в сети
Выполним команду __*show isis neighbors*__ на устройстве Spine1.
```
Spine1#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   C
100       default  Leaf1            L1L2 Ethernet1          P2P               UP    30          0
100       default  Leaf2            L1L2 Ethernet2          P2P               UP    26          0
100       default  Leaf3            L1L2 Ethernet3          P2P               UP    26          0
Spine1#
```
Вывод команды отображает все отношения соседства.

### Проверка IP связности устройств
Для проверки выполним команду __*ping*__ c устройства Spine1 до Spine2. В качестве source укажем loopback1:
```
Spine1#ping 10.0.2.0 source loopback 1
PING 10.0.2.0 (10.0.2.0) from 10.0.1.0 : 72(100) bytes of data.
80 bytes from 10.0.2.0: icmp_seq=1 ttl=63 time=28.2 ms
80 bytes from 10.0.2.0: icmp_seq=2 ttl=63 time=18.1 ms
80 bytes from 10.0.2.0: icmp_seq=3 ttl=63 time=8.52 ms
80 bytes from 10.0.2.0: icmp_seq=4 ttl=63 time=5.40 ms
80 bytes from 10.0.2.0: icmp_seq=5 ttl=63 time=5.22 ms

--- 10.0.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 70ms
rtt min/avg/max/mdev = 5.220/13.101/28.230/8.905 ms, pipe 3, ipg/ewma 17.594/20.125 ms
Spine1#
```
IP связность с устройством Spine2 есть.

Для проверки таблицы маршрутизации IS-IS выполним команду __*show ip route isis*.__

```
Spine1#show ip route isis

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

 I L1     10.0.0.1/32 [115/20] via 10.2.1.1, Ethernet1
 I L1     10.0.0.2/32 [115/20] via 10.2.1.3, Ethernet2
 I L1     10.0.0.3/32 [115/20] via 10.2.1.5, Ethernet3
 I L1     10.0.2.0/32 [115/30] via 10.2.1.1, Ethernet1
                               via 10.2.1.3, Ethernet2
                               via 10.2.1.5, Ethernet3
 I L1     10.2.2.0/31 [115/20] via 10.2.1.1, Ethernet1
 I L1     10.2.2.2/31 [115/20] via 10.2.1.3, Ethernet2
 I L1     10.2.2.4/31 [115/20] via 10.2.1.5, Ethernet3

Spine1#
```
Как видно из вывода таблицы маршрутизации все подсети успешно изучены.

IP связность между Leaf1 и Leaf3:
```
Leaf1#ping 10.0.0.3 source loopback 1
PING 10.0.0.3 (10.0.0.3) from 10.0.0.1 : 72(100) bytes of data.
80 bytes from 10.0.0.3: icmp_seq=1 ttl=63 time=8.98 ms
80 bytes from 10.0.0.3: icmp_seq=2 ttl=63 time=5.31 ms
80 bytes from 10.0.0.3: icmp_seq=3 ttl=63 time=4.17 ms
80 bytes from 10.0.0.3: icmp_seq=4 ttl=63 time=4.10 ms
80 bytes from 10.0.0.3: icmp_seq=5 ttl=63 time=4.60 ms

--- 10.0.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 34ms
rtt min/avg/max/mdev = 4.101/5.436/8.982/1.824 ms, ipg/ewma 8.613/7.134 ms
Leaf1#
```
IP связность между Leaf2 и Leaf3.
```
Leaf2#ping 10.0.0.3 source loopback 1
PING 10.0.0.3 (10.0.0.3) from 10.0.0.2 : 72(100) bytes of data.
80 bytes from 10.0.0.3: icmp_seq=1 ttl=63 time=19.0 ms
80 bytes from 10.0.0.3: icmp_seq=2 ttl=63 time=8.60 ms
80 bytes from 10.0.0.3: icmp_seq=3 ttl=63 time=8.62 ms
80 bytes from 10.0.0.3: icmp_seq=4 ttl=63 time=5.08 ms
80 bytes from 10.0.0.3: icmp_seq=5 ttl=63 time=4.52 ms

--- 10.0.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 65ms
rtt min/avg/max/mdev = 4.526/9.175/19.040/5.222 ms, pipe 2, ipg/ewma 16.441/13.s
Leaf2#
```



