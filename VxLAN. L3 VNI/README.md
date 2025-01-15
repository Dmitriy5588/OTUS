# Настройка VxLAN. eBGP_EVPN_L3VPN
### Цель:
- Настроить eBGP для Underlay сети;
- Настроить Overlay на основе VxLAN EVPN для L3 связанности между PC устройствами
### Описание сети
Топология сети и адреса сетевых устройств для Underlay сети были ранее сконфигурированны на лабораторном стенде.
Сетевые устройства будут поделены на автномные системы (BGP autonomous system, AS). Два устройства Spine, принадлежат одной AS65001, каждое устройства Leaf принадлежит своей AS (AS65002, AS65003, AS65004).
### Схема сети
![](https://github.com/Dmitriy5588/OTUS/blob/main/VxLAN.%20L3%20VNI/Screenshot_1.png)

В данной статье для простоты настроек будем ссылаться на уже выполенную часть в статье про [VxLAN L2 VNI](https://github.com/Dmitriy5588/OTUS/tree/main/VxLAN.%20L2%20VNI), поэтому мы просто добавим к уже готовой конфигурации дополнительные настройки для проверки L3 связности между устройствами.

Для L3 роутинга между устройствами в VxLAN фабрике, на Leaf коммутаторах необходимо:
- настроить VRF ( в нашем стенде __Test_L3VPN__)
- включить роутинг для VRF
- настроить VLAN SVI для наших подсетей и добавить в VRF
- настроить дополнительный SVI для L3VNI, добавить его в VRF, назначить ip адрес
- добавить наш VRF к VNI (в нашем стенде используем __5000__)
- добавить VRF в настройки BGP

### Настройка сетевых устройств 

Для теста выполним настройки только на двух Leaf коммутаторах, т.к. конфигурация будет схожей и для остальных Leaf.

#### Настройки на Leaf1
```
!
vrf instance Test_L3VPN
!
```
```
!
ip routing vrf Test_L3VPN
!
```
Настройки SVI
```
!
interface Vlan5
   vrf Test_L3VPN
   ip address virtual 10.10.10.1/24
!
interface Vlan20
   vrf Test_L3VPN
   ip address 192.168.2.1/24
!
interface Vlan30
   vrf Test_L3VPN
   ip address 192.168.31.1/24
!
```
Настройки интерфейса VxLAN
```
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 20 vni 2000
   vxlan vlan 30 vni 3000
   vxlan vrf Test_L3VPN vni 5000
!
```
Дополнительные настройки в блоке BGP
```
!
router bgp 65002
   !
   vrf Test_L3VPN
      rd 10.0.0.1:5
      route-target import evpn 5:5000
      route-target export evpn 5:5000
      redistribute connected
!
```

<details>
  
<summary> Общий вид настроек на Leaf2 </summary>

```
!
vlan 5,10,20
!
vrf instance Test_L3VPN
!
interface Ethernet1
   description to_Spine1
   no switchport
   ip address 10.2.1.3/31
!
interface Ethernet2
   description to_Spine2
   no switchport
   ip address 10.2.2.3/31
!
interface Ethernet3
   switchport access vlan 10
!
interface Ethernet4
   switchport access vlan 20
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback1
   ip address 10.0.0.2/32
!
interface Management1
!
interface Vlan5
   vrf Test_L3VPN
   ip address virtual 10.10.10.1/24
!
interface Vlan10
   vrf Test_L3VPN
   ip address 192.168.0.1/24
!
interface Vlan20
   vrf Test_L3VPN
   ip address 192.168.2.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 1000
   vxlan vlan 20 vni 2000
   vxlan vrf Test_L3VPN vni 5000
!
ip routing
ip routing vrf Test_L3VPN
!
route-map BGP_REDISTRIBUTE_CONNECTED permit 10
   match interface Loopback1
!
router bgp 65003
   router-id 10.0.0.2
   no bgp default ipv4-unicast
   distance bgp 20 200 200
   neighbor EVPN peer group
   neighbor EVPN remote-as 65001
   neighbor EVPN send-community extended
   neighbor 10.2.1.2 peer group EVPN
   neighbor 10.2.2.2 peer group EVPN
   redistribute connected route-map BGP_REDISTRIBUTE_CONNECTED
   !
   vlan 10
      rd auto
      route-target both 10:1000
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 20:2000
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   address-family ipv4
      neighbor 10.2.1.2 activate
      neighbor 10.2.2.2 activate
   !
   vrf Test_L3VPN
      rd 10.0.0.2:5
      route-target import evpn 5:5000
      route-target export evpn 5:5000
      redistribute connected
!
end
Leaf2#
```
</details>

### Проверим наши настройки show командами на Leaf1
```
Leaf1#show ip route vrf Test_L3VPN

VRF: Test_L3VPN
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

Gateway of last resort is not set

 B E      192.168.0.0/24 [20/0] via VTEP 10.0.0.2 VNI 5000 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 C        192.168.2.0/24 is directly connected, Vlan20
 C        192.168.31.0/24 is directly connected, Vlan30

Leaf1#
```
```
Leaf1#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.0.0.1, local AS number 65002
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.0.2:5 ip-prefix 192.168.0.0/24
                                 10.0.0.2              -       100     0       65001 65003 i
 *        RD: 10.0.0.2:5 ip-prefix 192.168.0.0/24
                                 10.0.0.2              -       100     0       65001 65003 i
 * >      RD: 10.0.0.1:5 ip-prefix 192.168.2.0/24
                                 -                     -       -       0       i
 * >      RD: 10.0.0.2:5 ip-prefix 192.168.2.0/24
                                 10.0.0.2              -       100     0       65001 65003 i
 *        RD: 10.0.0.2:5 ip-prefix 192.168.2.0/24
                                 10.0.0.2              -       100     0       65001 65003 i
 * >      RD: 10.0.0.1:5 ip-prefix 192.168.31.0/24
                                 -                     -       -       0       i
Leaf1#
```

Как видно из вывода проверочных команд у нас появился маршрут до второго VTEP, с назначеным нами VNI 5000, также отобразились Type 5 Routes, маршруты пятого типа с IP префиксом.

Запустим ping между VPC находящимся во Vlan 30 на Leaf1 и VPC во Vlan 10 на Leaf2:
```
VPCS : 192.168.31.20 255.255.255.0 gateway 192.168.31.1

VPCS> ping 192.168.0.50

84 bytes from 192.168.0.50 icmp_seq=1 ttl=62 time=342.426 ms
84 bytes from 192.168.0.50 icmp_seq=2 ttl=62 time=26.800 ms
84 bytes from 192.168.0.50 icmp_seq=3 ttl=62 time=12.783 ms
84 bytes from 192.168.0.50 icmp_seq=4 ttl=62 time=25.650 ms
84 bytes from 192.168.0.50 icmp_seq=5 ttl=62 time=28.283 ms

VPCS>
```
Как видно из вывода L3 связность между устройствами которые находятся в разных подсетях есть.
