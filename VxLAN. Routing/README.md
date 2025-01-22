# VxLAN. Routing
### Цель:
- Реализовать маршрутизацию между "клиентами" разных VRF через EVPN route-type 5
### Схема сети
![](https://github.com/Dmitriy5588/OTUS/blob/main/VxLAN.%20Routing/Routing.png)

Всё оборудование было настроено раннее в предыдущих лабораторных работах, в данную схему внесём дополнительные конфигурации для двух __Leaf__ , а также добавим новый коммутатор __Router__, который будет выполнять роль граничного маршрутизатора и выполнять Route leaking.

Для роутинга между устройствами разных VRF в VxLAN фабрике, на Leaf коммутаторах необходимо:
- настроить VRF
- установить BGP соседство с новым устройством (Router), в каждой VRF

### Настройка сетевых устройств

Соедениение с "граничным маршрутизатором" будем выполнять двумя линками от Leaf2 и Leaf3, вариантов подключения может быть больше, для простоты выполним как на схеме.
#### Настройки на Leaf2
```
!
vrf instance Test2
!
```
За данным vrf будут находиться клиенты в Vlan20, подсети 192.168.2.0.24
```
!
interface Vlan20
   vrf Test2
   ip address 192.168.2.1/24
!
```
Линк в сторону "граничного маршрутизатора"
```
!
interface Ethernet5
   no switchport
   vrf Test2
   ip address 172.168.2.1/24
!
```
Адрес 172.168.2.1 для p2p соединения с устройством Router и установления соседства по BGP.

В настройки BGP добавляем нового peer, устройство Router, с AS 65000
```
!
   vrf Test2
      rd 2.2.2.2:2
      route-target import evpn 2:200
      route-target export evpn 2:200
      neighbor 172.168.2.2 remote-as 65000
      redistribute connected
      !
      address-family ipv4
         neighbor 172.168.2.2 activate
!
```
#### Настройки на Leaf3
<details>
  
<summary> Конфигурация Leaf3 </summary>

```
!
vrf instance Test1
!
vrf instance Test3
!
```
Подсети клиентов в Vlan10 и Vlan30
```
!
interface Vlan10
   vrf Test1
   ip address 192.168.0.1/24
!
interface Vlan30
   vrf Test3
   ip address 192.168.31.1/24
!
```
Линк в сторону Router настраиваем через sub интерфесы
```
!
interface Ethernet5
   no switchport
!
interface Ethernet5.101
   encapsulation dot1q vlan 101
   vrf Test1
   ip address 172.168.0.1/24
!
interface Ethernet5.103
   encapsulation dot1q vlan 103
   vrf Test3
   ip address 172.168.3.1/24
!
```
Дополнительные настройки в процесс BGP.
```
!
   vrf Test1
      rd 1.1.1.1:1
      route-target import evpn 1:100
      route-target export evpn 1:100
      neighbor 172.168.0.2 remote-as 65000
      redistribute connected
      !
      address-family ipv4
         neighbor 172.168.0.2 activate
   !
   vrf Test3
      rd 3.3.3.3:3
      route-target import evpn 3:300
      route-target export evpn 3:300
      neighbor 172.168.3.2 remote-as 65000
      redistribute connected
      !
      address-family ipv4
         neighbor 172.168.3.2 activate
!
```
</details>

#### Общий вид настроек на новом устройстве Router
```
!
interface Ethernet1
   no switchport
   ip address 172.168.2.2/24
!
interface Ethernet2
   no switchport
!
interface Ethernet2.101
   encapsulation dot1q vlan 101
   ip address 172.168.0.2/24
!
interface Ethernet2.103
   encapsulation dot1q vlan 103
   ip address 172.168.3.2/24
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
   no switchport
   ip address 50.1.1.2/24
!
interface Management1
!
ip routing
!
ip route 0.0.0.0/24 Ethernet8
!
router bgp 65000
   router-id 10.10.10.1
   distance bgp 20 200 200
   neighbor 172.168.0.1 remote-as 65004
   neighbor 172.168.0.1 default-originate
   neighbor 172.168.2.1 remote-as 65003
   neighbor 172.168.2.1 default-originate
   neighbor 172.168.3.1 remote-as 65004
   neighbor 172.168.3.1 default-originate
!
end
Router#

Interface Ethernet8 для связи с условным роутером провайдера, дефолтный маршрут для выхода из Vxlan фабрики.

```

Запустим ping между VPC находящимся во Vlan 10 на Leaf3 и VPC во Vlan 20 на Leaf2, в vrf Test1 и vrf Test2:
```
VPCS> show ip

NAME        : VPCS[1]
IP/MASK     : 192.168.0.101/24
GATEWAY     : 192.168.0.1
DNS         :
MAC         : 00:50:79:66:68:0b
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPCS> ping 192.168.2.100

84 bytes from 192.168.2.100 icmp_seq=1 ttl=61 time=110.467 ms
84 bytes from 192.168.2.100 icmp_seq=2 ttl=61 time=10.233 ms
84 bytes from 192.168.2.100 icmp_seq=3 ttl=61 time=11.736 ms
84 bytes from 192.168.2.100 icmp_seq=4 ttl=61 time=19.236 ms
84 bytes from 192.168.2.100 icmp_seq=5 ttl=61 time=12.056 ms

VPCS>
```
Связность между устройствами в разных vrf есть.

Для примера проверим дефолтный маршрут на Leaf3
```
Leaf3#show ip route vrf Test3

VRF: Test3
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

Gateway of last resort:
 B E      0.0.0.0/0 [20/0] via 172.168.3.2, Ethernet5.103

 B E      172.168.2.0/24 [20/0] via 172.168.3.2, Ethernet5.103
 C        172.168.3.0/24 is directly connected, Ethernet5.103
 B E      192.168.2.0/24 [20/0] via 172.168.3.2, Ethernet5.103
 C        192.168.31.0/24 is directly connected, Vlan30

Leaf3#
```

  
