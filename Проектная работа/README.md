# Проектная работа

## Тема: Проектирование сети двух территориально разнесённых ЦОД с применением технологии VxLAN/EVPN.
### Цель:
- Определить архитектуру сети
- Сформировать адресное пространство
- Сконфигурировать Underlay и Overlay сети
- Настроить DCI (Data Center Interconnection) между площадками

## Описание
В данном проектном решении необходимо спроектировать и настроить базовую сеть двух территориально разнесённых ЦОД, основной и резервный, с возможностью дальнейшего масштабирования. Схема будущего расширения на рисунке ниже.
![](https://github.com/Dmitriy5588/OTUS/blob/main/%D0%9F%D1%80%D0%BE%D0%B5%D0%BA%D1%82%D0%BD%D0%B0%D1%8F%20%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%B0/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D0%BC%D0%B0%D1%81%D1%88%D1%82%D0%B0%D0%B1%D0%B8%D1%80%D1%83%D0%B5%D0%BC%D0%BE%D0%B9%20%D1%81%D0%B5%D1%82%D0%B8.png)

Архитектура ЦОД будет построена по современной модели сети CLOS, которая позволяет собрать неблокируемую сетевую топологию, состоящую из нескольких уровней, Leaf и Spine. Underlay построена на базе протокола OSPF, причём основной ЦОД и резервный ЦОД находятся в разных AREA, DCI является Backbone AREA. Overlay основного ЦОД и резервного ЦОД работает по протоколу iBGP в разных AS, Spine принимает роль Route-reflector (RR), на DCI настроен eBGP.
Для объединение площадок ЦОД применим вариант архитектуры Multi-pod.

### Схема сети
![](https://github.com/Dmitriy5588/OTUS/blob/main/%D0%9F%D1%80%D0%BE%D0%B5%D0%BA%D1%82%D0%BD%D0%B0%D1%8F%20%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%B0/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D0%B5%D1%82%D0%B8.png)

### Адресное пространство
|_Device_|_Port_|_IP Address_|_Subnet_ _Mask_|_Link_
|---|---|---|---|---|
Spine1|Eth1|10.1.1.1|255.255.255.254|p2p
Spine1|Eth2|10.1.2.1|255.255.255.254|p2p
Spine1|Eth3|10.1.3.1|255.255.255.254|p2p
Spine1|*__Lo1__*|11.11.11.11|255.255.255.255|Loopback1
Leaf1|Eth1|10.1.1.2|255.255.255.254|p2p
Leaf1|*__Lo1__*|1.1.1.1|255.255.255.255|Loopback1
Leaf2|Eth1|10.1.2.2|255.255.255.254|p2p
Leaf2|*__Lo1__*|1.1.1.2|255.255.255.255|Loopback1
Border_Leaf2|Eth1|10.1.3.2|255.255.255.254|p2p
Border_Leaf2|Eth7|100.1.1.1|255.255.255.254|p2p
Border_Leaf2|Eth8|100.1.2.1|255.255.255.254|p2p
Border_Leaf2|*__Lo1__*|1.1.1.3|255.255.255.255|Loopback1
|---|---|---|---|---|
Spine2-1|Eth1|20.1.1.1|255.255.255.254|p2p
Spine2-1|Eth3|20.1.3.1|255.255.255.254|p2p
Spine2-1|*__Lo1__*|22.22.22.22|255.255.255.255|Loopback1
Leaf2-1|Eth1|20.1.1.2|255.255.255.254|p2p
Leaf2-1|*__Lo1__*|2.2.2.1|255.255.255.255|Loopback1
Border_Leaf2-1|Eth1|20.1.3.2|255.255.255.254|p2p
Border_Leaf2-1|Eth7|100.1.1.2|255.255.255.254|p2p
Border_Leaf2-1|Eth8|100.1.2.2|255.255.255.254|p2p
Border_Leaf2-1|*__Lo1__*|2.2.2.3|255.255.255.255|Loopback1

### Настройка сетевых устройств
#### Настройка Underlay
<details>
  
<summary> Настройки на Spine1  </summary>

```
!
interface Ethernet1
   description to_Leaf1
   no switchport
   ip address 10.1.1.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.1
!
interface Ethernet2
   description to_Leaf2
   no switchport
   ip address 10.1.2.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.1
!
interface Ethernet3
   description to_Border_Leaf1
   no switchport
   ip address 10.1.3.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.1
!
!
interface Loopback1
   ip address 11.11.11.11/32
   ip ospf area 0.0.0.1
!
!
router ospf 1
   router-id 11.11.11.11
   bfd default
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
!
end
Spine1
```
</details>

<details>
  
<summary> Настройки на Leaf1  </summary>

```
!
interface Ethernet1
   description to_Spine1
   no switchport
   ip address 10.1.1.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.1
!
!
interface Loopback1
   ip address 1.1.1.1/32
   ip ospf area 0.0.0.1
!
!
router ospf 1
   router-id 1.1.1.1
   bfd default
   passive-interface default
   no passive-interface Ethernet1
   max-lsa 12000
!
```
</details>

<details>
  
<summary> Настройки на Leaf2  </summary>

```
!
interface Ethernet1
   description to_Spine1
   no switchport
   ip address 10.1.2.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.1
!
!
interface Loopback1
   ip address 1.1.1.2/32
   ip ospf area 0.0.0.1
!
!
router ospf 1
   router-id 1.1.1.2
   bfd default
   passive-interface default
   no passive-interface Ethernet1
   max-lsa 12000
!
end
Leaf2#
```
</details>

<details>
  
<summary> Настройки на Border_Leaf1  </summary>

```
!
interface Ethernet1
   description to_Spine1
   no switchport
   ip address 10.1.3.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.1
!
!
interface Ethernet7
   description to_Border_Leaf2-1
   no switchport
   ip address 100.1.1.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet8
   description to_Border_Leaf2-1
   no switchport
   ip address 100.1.2.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 1.1.1.3/32
   ip ospf area 0.0.0.1
!
!
router ospf 1
   router-id 1.1.1.3
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet7
   no passive-interface Ethernet8
   max-lsa 12000
!
end
Border_Leaf1#
```
</details>

<details>
  
<summary> Настройки на Border_Leaf2-1 </summary>

```
!
interface Ethernet1
   description to_Spine2-1
   no switchport
   ip address 20.1.3.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.2
!
!
interface Ethernet7
   description to_Border_Leaf1
   no switchport
   ip address 100.1.1.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet8
   description to_Border_Leaf1
   no switchport
   ip address 100.1.2.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 2.2.2.3/32
   ip ospf area 0.0.0.2
!
!
router ospf 1
   router-id 2.2.2.3
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet7
   no passive-interface Ethernet8
   max-lsa 12000
!
end
Border_Leaf2-1#
```
</details>

<details>
  
<summary> Настройки на Spine2-1 </summary>

```
!
interface Ethernet1
   description to_Leaf2-1
   no switchport
   ip address 20.1.1.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.2
!
!
interface Loopback1
   ip address 22.22.22.22/32
   ip ospf area 0.0.0.2
!
!
router ospf 1
   router-id 22.22.22.22
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
!
end
Spine2-1#
```
</details>

<details>
  
<summary> Настройки на Leaf2-1 </summary>

```
!
interface Ethernet1
   description to_Spine2-1
   no switchport
   ip address 20.1.1.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.2
!
!
interface Loopback1
   ip address 2.2.2.1/32
   ip ospf area 0.0.0.2
!
!
router ospf 1
   router-id 2.2.2.1
   passive-interface default
   no passive-interface Ethernet1
   max-lsa 12000
!
end
Leaf2-1#
```
</details>

#### Настройка Overlay VxLAN BGP/EVPN
<details>
  
<summary> Настройки на Spine1 </summary>

```
!
router bgp 65001
   router-id 11.11.11.11
   no bgp default ipv4-unicast
   distance bgp 20 200 200
   neighbor EVPN peer group
   neighbor EVPN remote-as 65001
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback1
   neighbor EVPN route-reflector-client
   neighbor EVPN password 7 sFgdj+LZq58=
   neighbor EVPN send-community extended
   neighbor 1.1.1.1 peer group EVPN
   neighbor 1.1.1.2 peer group EVPN
   neighbor 1.1.1.3 peer group EVPN
   neighbor 22.22.22.22 remote-as 65002
   neighbor 22.22.22.22 update-source Loopback1
   neighbor 22.22.22.22 ebgp-multihop 4
   neighbor 22.22.22.22 send-community extended
   !
   address-family evpn
      neighbor EVPN activate
      neighbor 22.22.22.22 activate
!
```
</details>

<details>
  
<summary> Настройки на Leaf1 </summary>

```
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 1010
   vxlan vlan 20 vni 1020
!
ip routing
!
router bgp 65001
   router-id 1.1.1.1
   no bgp default ipv4-unicast
   distance bgp 20 200 200
   neighbor 11.11.11.11 remote-as 65001
   neighbor 11.11.11.11 update-source Loopback1
   neighbor 11.11.11.11 password 7 K+qPl7W8934=
   neighbor 11.11.11.11 send-community extended
   !
   vlan 10
      rd auto
      route-target both 65001:10
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 65001:20
      redistribute learned
   !
   address-family evpn
      neighbor 11.11.11.11 activate
!
```
</details>

<details>
  
<summary> Настройки на Leaf2 </summary>

```
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 1010
   vxlan vlan 20 vni 1020
!
ip routing
!
router bgp 65001
   router-id 1.1.1.2
   no bgp default ipv4-unicast
   distance bgp 20 200 200
   neighbor 11.11.11.11 remote-as 65001
   neighbor 11.11.11.11 update-source Loopback1
   neighbor 11.11.11.11 password 7 K+qPl7W8934=
   neighbor 11.11.11.11 send-community extended
   !
   vlan 10
      rd auto
      route-target both 65001:10
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 65001:20
      redistribute learned
   !
   address-family evpn
      neighbor 11.11.11.11 activate
!
```
</details>

<details>
  
<summary> Настройки на Border_Leaf1 </summary>

```
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 1010
   vxlan vlan 20 vni 1020
!
ip routing
!
router bgp 65001
   router-id 1.1.1.3
   no bgp default ipv4-unicast
   distance bgp 20 200 200
   neighbor 11.11.11.11 remote-as 65001
   neighbor 11.11.11.11 update-source Loopback1
   neighbor 11.11.11.11 password 7 K+qPl7W8934=
   neighbor 11.11.11.11 send-community extended
   !
   vlan 10
      rd auto
      route-target both 65001:10
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 65001:20
      redistribute learned
   !
   address-family evpn
      neighbor 11.11.11.11 activate
!
```
</details>

<details>
  
<summary> Настройки на Border_Leaf2-1 </summary>

```
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 1010
   vxlan vlan 20 vni 1020
!
ip routing
!
router bgp 65002
   router-id 2.2.2.3
   no bgp default ipv4-unicast
   distance bgp 20 200 200
   neighbor 22.22.22.22 remote-as 65002
   neighbor 22.22.22.22 update-source Loopback1
   neighbor 22.22.22.22 password 7 URaCpeObwYU=
   neighbor 22.22.22.22 send-community extended
   !
   vlan 10
      rd auto
      route-target both 65001:10
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 65001:20
      redistribute learned
   !
   address-family evpn
      neighbor 22.22.22.22 activate
!
```
</details>

<details>
  
<summary> Настройки на Spine2-1 </summary>

```
!
router bgp 65002
   router-id 22.22.22.22
   no bgp default ipv4-unicast
   distance bgp 20 200 200
   neighbor EVPN peer group
   neighbor EVPN remote-as 65002
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback1
   neighbor EVPN route-reflector-client
   neighbor EVPN password 7 sFgdj+LZq58=
   neighbor EVPN send-community extended
   neighbor 2.2.2.1 peer group EVPN
   neighbor 2.2.2.3 peer group EVPN
   neighbor 11.11.11.11 remote-as 65001
   neighbor 11.11.11.11 update-source Loopback1
   neighbor 11.11.11.11 ebgp-multihop 4
   neighbor 11.11.11.11 send-community extended
   !
   address-family evpn
      neighbor EVPN activate
      neighbor 11.11.11.11 activate
!
```
</details>

<details>
  
<summary> Настройки на Leaf2-1 </summary>

```
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 1010
   vxlan vlan 20 vni 1020
!
ip routing
!
router bgp 65002
   router-id 2.2.2.1
   no bgp default ipv4-unicast
   distance bgp 20 200 200
   neighbor 22.22.22.22 remote-as 65002
   neighbor 22.22.22.22 update-source Loopback1
   neighbor 22.22.22.22 password 7 URaCpeObwYU=
   neighbor 22.22.22.22 send-community extended
   !
   vlan 10
      rd auto
      route-target both 65001:10
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 65001:20
      redistribute learned
   !
   address-family evpn
      neighbor 22.22.22.22 activate
!
```
</details>

#### Логическая связка DCI проходит между ЦОД на устройствах Spine1 и Spine2-1, по протоколу eBGP. Физическое подключение выполнено на устройствах Border_Leaf двумя L3 линками для отказоустойчивости.

### Проверка работы Vxlan фабрики
#### Обнаружение соседств и связности на устройстве Leaf1 в основном ЦОД
```
Leaf1#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP          Tunnel Type(s)
------------- --------------
1.1.1.2       flood
1.1.1.3       flood
2.2.2.1       flood
2.2.2.3       flood

Total number of remote VTEPS:  4
Leaf1#
```
```
Leaf1#show ip ospf neighbor
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
11.11.11.11     1        default  0   FULL                   00:00:36    10.1.1.1        Ethernet1
Leaf1#show ip route ospf

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

 O        1.1.1.2/32 [110/30] via 10.1.1.1, Ethernet1
 O        1.1.1.3/32 [110/30] via 10.1.1.1, Ethernet1
 O IA     2.2.2.1/32 [110/60] via 10.1.1.1, Ethernet1
 O IA     2.2.2.3/32 [110/40] via 10.1.1.1, Ethernet1
 O        10.1.2.0/30 [110/20] via 10.1.1.1, Ethernet1
 O        10.1.3.0/30 [110/20] via 10.1.1.1, Ethernet1
 O        11.11.11.11/32 [110/20] via 10.1.1.1, Ethernet1
 O IA     20.1.1.0/30 [110/50] via 10.1.1.1, Ethernet1
 O IA     20.1.3.0/30 [110/40] via 10.1.1.1, Ethernet1
 O IA     22.22.22.22/32 [110/50] via 10.1.1.1, Ethernet1
 O IA     100.1.1.0/30 [110/30] via 10.1.1.1, Ethernet1
 O IA     100.1.2.0/30 [110/30] via 10.1.1.1, Ethernet1

Leaf1#
```
Все маршруты в Underlay сети находятся в таблице.

#### Обнаружение соседств и связности на устройстве Leaf2-1 в резервном ЦОД
```
Leaf2-1#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP          Tunnel Type(s)
------------- --------------
1.1.1.1       flood
1.1.1.2       flood
1.1.1.3       flood
2.2.2.3       flood

Total number of remote VTEPS:  4
```
```
Leaf2-1#show ip route ospf

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

 O IA     1.1.1.1/32 [110/60] via 20.1.1.1, Ethernet1
 O IA     1.1.1.2/32 [110/60] via 20.1.1.1, Ethernet1
 O IA     1.1.1.3/32 [110/40] via 20.1.1.1, Ethernet1
 O        2.2.2.3/32 [110/30] via 20.1.1.1, Ethernet1
 O IA     10.1.1.0/30 [110/50] via 20.1.1.1, Ethernet1
 O IA     10.1.2.0/30 [110/50] via 20.1.1.1, Ethernet1
 O IA     10.1.3.0/30 [110/40] via 20.1.1.1, Ethernet1
 O IA     11.11.11.11/32 [110/50] via 20.1.1.1, Ethernet1
 O        20.1.3.0/30 [110/20] via 20.1.1.1, Ethernet1
 O        22.22.22.22/32 [110/20] via 20.1.1.1, Ethernet1
 O IA     100.1.1.0/30 [110/30] via 20.1.1.1, Ethernet1
 O IA     100.1.2.0/30 [110/30] via 20.1.1.1, Ethernet1

Leaf2-1#
```

#### Для примера выполним команду ping из основного ЦОД в резервный, host1 - host3.
```
VPCS> ping 192.168.0.200

84 bytes from 192.168.0.200 icmp_seq=1 ttl=64 time=52.216 ms
84 bytes from 192.168.0.200 icmp_seq=2 ttl=64 time=33.975 ms
84 bytes from 192.168.0.200 icmp_seq=3 ttl=64 time=58.426 ms
84 bytes from 192.168.0.200 icmp_seq=4 ttl=64 time=31.495 ms
84 bytes from 192.168.0.200 icmp_seq=5 ttl=64 time=27.188 ms

VPCS>
```
#### Вывод таблициы EVPN на примере Leaf1
```
Leaf1#show bgp evpn
BGP routing table information for VRF default
Router identifier 1.1.1.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 1.1.1.1:10 mac-ip 0050.7966.6807
                                 -                     -       -       0       i
 * >      RD: 1.1.1.1:10 mac-ip 0050.7966.6807 192.168.0.100
                                 -                     -       -       0       i
 * >      RD: 2.2.2.1:10 mac-ip 0050.7966.680a
                                 2.2.2.1               -       100     0       65002 i
 * >      RD: 1.1.1.1:10 imet 1.1.1.1
                                 -                     -       -       0       i
 * >      RD: 1.1.1.1:20 imet 1.1.1.1
                                 -                     -       -       0       i
 * >      RD: 1.1.1.2:10 imet 1.1.1.2
                                 1.1.1.2               -       100     0       i Or-ID: 1.1.1.2 C-LST: 11.11.11.11
 * >      RD: 1.1.1.2:20 imet 1.1.1.2
                                 1.1.1.2               -       100     0       i Or-ID: 1.1.1.2 C-LST: 11.11.11.11
 * >      RD: 1.1.1.3:10 imet 1.1.1.3
                                 1.1.1.3               -       100     0       i Or-ID: 1.1.1.3 C-LST: 11.11.11.11
 * >      RD: 1.1.1.3:20 imet 1.1.1.3
                                 1.1.1.3               -       100     0       i Or-ID: 1.1.1.3 C-LST: 11.11.11.11
 * >      RD: 2.2.2.1:10 imet 2.2.2.1
                                 2.2.2.1               -       100     0       65002 i
 * >      RD: 2.2.2.1:20 imet 2.2.2.1
                                 2.2.2.1               -       100     0       65002 i
 * >      RD: 2.2.2.3:10 imet 2.2.2.3
                                 2.2.2.3               -       100     0       65002 i
 * >      RD: 2.2.2.3:20 imet 2.2.2.3
                                 2.2.2.3               -       100     0       65002 i
Leaf1#
```

### Делаем вывод, Vxlan фабрика между ЦОД собрана, дальнейшее развитие согласно схеме выше выполнимо.

