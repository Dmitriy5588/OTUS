# Настройка VxLAN. eBGP_EVPN_L2VPN
### Цель:
- Настроить eBGP для Underlay сети;
- Настроить Overlay на основе VxLAN EVPN для L2 связанности между PC устройствами
### Краткое описание протокола
__VXLAN__ — это технология или протокол туннелирования, который инкапсулирует L2 кадры внутри UDP-пакетов, обычно отправляемых на порт 4789.

Он обеспечивает:
 - Туннелирование L2 поверх L3, чтобы избежать необходимость организовывать L2 связность между всеми хостами в кластере
 - Более 4096 изолированных сетей (идентификаторы VLAN ограничены 4096)
 - Использование многоадресной рассылки в транспортной сети для имитации поведения лавинной рассылки в широковещательной, неизвестной одноадресной и многоадресной рассылке в сегменте L2
 - Использование выбора маршрута в зависимости от стоимости (ECMP) для обеспечения использования оптимального пути по транспортной сети
   
Некоторые основные термины VxLAN:
- __VXLAN__ (Virtual eXtensible Local Area Network) — расширяемая виртуальная ЛВС;
- __VNI__ (VXLAN Network Identifier) — идентификатор сети VXLAN, номер логической сети (аналог VLAN id);
- __VTEP__ (VXLAN Tunnel End Point) — конечная точка туннеля VXLAN, коммутаторы, которые терминируют туннель VXLAN;
- __NVE__ (Network Virtual Edge) — сущность самого туннеля, то, как реализован сам туннель.

__BGP EVPN__ — это стандартный протокол управления, основанный на протоколе BGP (Border Gateway Protocol) и его расширениях MP-BGP.

__EVPN__ — это расширение BGP, которое позволяет передавать информацию о доступности сети для MAC-адресов и удалённого оборудования. 

### Описание сети
Топология сети и адреса сетевых устройств для Underlay сети были ранее сконфигурированны на лабораторном стенде.
Сетевые устройства будут поделены на автномные системы (BGP autonomous system, AS). Два устройства Spine, принадлежат одной AS65001, каждое устройства Leaf принадлежит своей AS (AS65002, AS65003, AS65004).

### Схема сети
![](https://github.com/Dmitriy5588/OTUS/blob/main/VxLAN.%20L2%20VNI/VxLAN_eBGP_EVPN_L2VPN_.png)

### Таблица адресов

|_Device_|_Port_|_IP Address_|_Subnet_ _Mask_|_Port_
|---|---|---|---|---|
Spine1|Eth1|10.2.1.0|255.255.255.254|Eth1
Spine1|Eth2|10.2.1.2|255.255.255.254|Eth2
Spine1|Eth3|10.2.1.4|255.255.255.254|Eth3
**Spine1**|*__Lo1__*|10.0.1.0|255.255.255.255|Loopback1
Spine2|Eth1|10.2.2.0|255.255.255.254|Eth1
Spine2|Eth2|10.2.2.2|255.255.255.254|Eth2
Spine2|Eth3|10.2.2.4|255.255.255.254|Eth3
**Spine2**|*__Lo1__*|10.0.2.0|255.255.255.255|Loopback1
Leaf1|Eth1|10.2.1.1|255.255.255.254|Eth1
Leaf1|Eth2|10.2.2.1|255.255.255.254|Eth2
**Leaf1**|*__Lo1__*|10.0.0.1|255.255.255.255|Loopback1
Leaf2|Eth1|10.2.1.3|255.255.255.254|Eth1
Leaf2|Eth2|10.2.2.3|255.255.255.254|Eth2
**Leaf2**|*__Lo1__*|10.0.0.2|255.255.255.255|Loopback1
Leaf3|Eth1|10.2.1.5|255.255.255.254|Eth1
Leaf3|Eth2|10.2.2.5|255.255.255.254|Eth2
**Leaf3**|*__Lo1__*|10.0.0.3|255.255.255.255|Loopback1

<details>

<summary> VLAN Таблица </summary>

|VLAN|IP address/mask|Связанные PC|
|---|---|---|
10|192.168.0.50/24|VPC#3
10|192.168.0.101/24|VPC#6
20|192.168.2.200/24|VPC#2
20|192.168.2.100/24|VPC#4
30|192.168.31.20/24|VPC#1
30|192.168.31.10/24|VPC#5

</details>


### Настройка сетевых устройств
Т.к. устройства Spine в нашей топологии служат  для пересылки пакетов и не участвуют в процессе икапсуляции/деинкапсуляции VxLAN, то на них выполним только настройки eBGP, и настройки для работы BGP address-family EVPN.

На всех устройствах нашей сети создадим Route-map для анонсирования только loopback адресов.

#### Настройка BGP на Spine1 и Spine2
```
!
route-map BGP_REDISTRIBUTE_CONNECTED permit 10
   match interface Loopback1
!
```
Настройки BGP EVPN:
<details>
  
<summary> Настройки на Spine1 </summary>

```
!
router bgp 65001
   router-id 10.0.1.0
   no bgp default ipv4-unicast
   distance bgp 20 200 200
   neighbor EVPN peer group
   neighbor EVPN send-community extended
   neighbor 10.2.1.1 peer group EVPN
   neighbor 10.2.1.1 remote-as 65002
   neighbor 10.2.1.3 peer group EVPN
   neighbor 10.2.1.3 remote-as 65003
   neighbor 10.2.1.5 peer group EVPN
   neighbor 10.2.1.5 remote-as 65004
   redistribute connected route-map BGP_REDISTRIBUTE_CONNECTED
   !
   address-family evpn
      neighbor EVPN activate
   !
   address-family ipv4
      neighbor 10.2.1.1 activate
      neighbor 10.2.1.3 activate
      neighbor 10.2.1.5 activate
!
```
</details>

<details>
  
<summary> Настройки на Spine2 </summary>

```
!
router bgp 65001
   router-id 10.0.1.0
   no bgp default ipv4-unicast
   distance bgp 20 200 200
   neighbor EVPN peer group
   neighbor EVPN send-community extended
   neighbor 10.2.2.1 peer group EVPN
   neighbor 10.2.2.1 remote-as 65002
   neighbor 10.2.2.3 peer group EVPN
   neighbor 10.2.2.3 remote-as 65003
   neighbor 10.2.2.5 peer group EVPN
   neighbor 10.2.2.5 remote-as 65004
   redistribute connected route-map BGP_REDISTRIBUTE_CONNECTED
   !
   address-family evpn
      neighbor EVPN activate
   !
   address-family ipv4
      neighbor 10.2.2.1 activate
      neighbor 10.2.2.3 activate
      neighbor 10.2.2.5 activate
!
```
</details>

### Настройки на Leaf1, Leaf2, Leaf3

На каждом Leaf добавим Vlan согласно схеме нашей фабрики, настроим access порты для подключения VPC.

<details>
  
<summary> Настройки на Leaf1 </summary>

```
!
interface Ethernet3
   switchport access vlan 30
!
interface Ethernet4
   switchport access vlan 20
!
```
</details>

<details>
  
<summary> Настройки на Leaf2 </summary>

```
!
interface Ethernet3
   switchport access vlan 10
!
interface Ethernet4
   switchport access vlan 20
!

```
</details>

<details>
  
<summary> Настройки на Leaf3 </summary>

```
!
interface Ethernet3
   switchport access vlan 30
!
interface Ethernet4
   switchport access vlan 10
!
```
</details>

#### Переходим к настройкам VxLAN и EVPN L2VPN
В нашей архитектуре каждый Vlan будет сопоставляться своему VNI.

В качестве source-interface настроим Loopback1, это будет адрес VTEP, внешний Source IP в VXLAN-пакетах.
<details>
  
<summary> Настройки на Leaf1 </summary>

```
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 20 vni 2000
   vxlan vlan 30 vni 3000
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
   vxlan vlan 10 vni 1000
   vxlan vlan 20 vni 2000
!
```
</details>

<details>
  
<summary> Настройки на Leaf3 </summary>

```
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 1000
   vxlan vlan 30 vni 3000
!
```
</details>

#### Настройки для работы в сервисе Layer 2
В процесс BGP добавляем конфигурацию наших vlan, RD, RT, и способ изучения Ethernet адресов.

<details>
  
<summary> Настройки на Leaf1 </summary>

```
!
router bgp 65002
   router-id 10.0.0.1
   no bgp default ipv4-unicast
   distance bgp 20 200 200
   neighbor EVPN peer group
   neighbor EVPN remote-as 65001
   neighbor EVPN send-community extended
   neighbor 10.2.1.0 peer group EVPN
   neighbor 10.2.2.0 peer group EVPN
   redistribute connected route-map BGP_REDISTRIBUTE_CONNECTED
   !
   vlan 20
      rd auto
      route-target both 20:2000
      redistribute learned
   !
   vlan 30
      rd auto
      route-target both 30:3000
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   address-family ipv4
      neighbor 10.2.1.0 activate
      neighbor 10.2.2.0 activate
!
```
</details>

<details>
  
<summary> Настройки на Leaf2 </summary>

```
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
```
</details>

<details>
  
<summary> Настройки на Leaf3 </summary>

```
!
router bgp 65004
   router-id 10.0.0.3
   no bgp default ipv4-unicast
   distance bgp 20 200 200
   neighbor EVPN peer group
   neighbor EVPN remote-as 65001
   neighbor EVPN send-community extended
   neighbor 10.2.1.4 peer group EVPN
   neighbor 10.2.2.4 peer group EVPN
   redistribute connected route-map BGP_REDISTRIBUTE_CONNECTED
   !
   vlan 10
      rd auto
      route-target both 10:1000
      redistribute learned
   !
   vlan 30
      rd auto
      route-target both 30:3000
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   address-family ipv4
      neighbor 10.2.1.4 activate
      neighbor 10.2.2.4 activate
!
```
</details>

#### IP адреса на VPC назначим согласно Vlan таблице.
### Итоговая проверка связанности между PC и таблиц MAC VRF
Запустим ping между VPC.
#### Между VPC#1 и VPC#5
```
VPCS> ping 192.168.31.10

84 bytes from 192.168.31.10 icmp_seq=1 ttl=64 time=26.403 ms
84 bytes from 192.168.31.10 icmp_seq=2 ttl=64 time=30.005 ms
84 bytes from 192.168.31.10 icmp_seq=3 ttl=64 time=14.327 ms
84 bytes from 192.168.31.10 icmp_seq=4 ttl=64 time=28.007 ms
84 bytes from 192.168.31.10 icmp_seq=5 ttl=64 time=16.103 ms
```

#### Между VPC#2 и VPC#4
```
VPCS> ping 192.168.2.100

84 bytes from 192.168.2.100 icmp_seq=1 ttl=64 time=16.002 ms
84 bytes from 192.168.2.100 icmp_seq=2 ttl=64 time=14.891 ms
84 bytes from 192.168.2.100 icmp_seq=3 ttl=64 time=30.729 ms
84 bytes from 192.168.2.100 icmp_seq=4 ttl=64 time=27.086 ms
84 bytes from 192.168.2.100 icmp_seq=5 ttl=64 time=27.754 ms
```
#### Между VPC#3 и VPC#6
```
VPCS> ping 192.168.0.101

84 bytes from 192.168.0.101 icmp_seq=1 ttl=64 time=16.388 ms
84 bytes from 192.168.0.101 icmp_seq=2 ttl=64 time=13.063 ms
84 bytes from 192.168.0.101 icmp_seq=3 ttl=64 time=14.670 ms
84 bytes from 192.168.0.101 icmp_seq=4 ttl=64 time=27.271 ms
84 bytes from 192.168.0.101 icmp_seq=5 ttl=64 time=12.746 ms
```

#### Проверим, какие маршруты мы получаем в AFI EVPN:
```
Leaf1#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.0.0.1, local AS number 65002
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.0.1:30 mac-ip 0050.7966.6806
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.0.2:10 mac-ip 0050.7966.6807
                                 10.0.0.2              -       100     0       65001 65003 i
 *  ec    RD: 10.0.0.2:10 mac-ip 0050.7966.6807
                                 10.0.0.2              -       100     0       65001 65003 i
 * >Ec    RD: 10.0.0.3:30 mac-ip 0050.7966.6808
                                 10.0.0.3              -       100     0       65001 65004 i
 *  ec    RD: 10.0.0.3:30 mac-ip 0050.7966.6808
                                 10.0.0.3              -       100     0       65001 65004 i
 * >      RD: 10.0.0.1:20 mac-ip 0050.7966.6809
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.0.2:20 mac-ip 0050.7966.680a
                                 10.0.0.2              -       100     0       65001 65003 i
 *  ec    RD: 10.0.0.2:20 mac-ip 0050.7966.680a
                                 10.0.0.2              -       100     0       65001 65003 i
 * >Ec    RD: 10.0.0.3:10 mac-ip 0050.7966.680b
                                 10.0.0.3              -       100     0       65001 65004 i
 *  ec    RD: 10.0.0.3:10 mac-ip 0050.7966.680b
                                 10.0.0.3              -       100     0       65001 65004 i
 * >      RD: 10.0.0.1:20 imet 10.0.0.1
                                 -                     -       -       0       i
 * >      RD: 10.0.0.1:30 imet 10.0.0.1
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.0.2:10 imet 10.0.0.2
                                 10.0.0.2              -       100     0       65001 65003 i
 *  ec    RD: 10.0.0.2:10 imet 10.0.0.2
                                 10.0.0.2              -       100     0       65001 65003 i
 * >Ec    RD: 10.0.0.2:20 imet 10.0.0.2
                                 10.0.0.2              -       100     0       65001 65003 i
 *  ec    RD: 10.0.0.2:20 imet 10.0.0.2
                                 10.0.0.2              -       100     0       65001 65003 i
 * >Ec    RD: 10.0.0.3:10 imet 10.0.0.3
                                 10.0.0.3              -       100     0       65001 65004 i
 *  ec    RD: 10.0.0.3:10 imet 10.0.0.3
                                 10.0.0.3              -       100     0       65001 65004 i
 * >Ec    RD: 10.0.0.3:30 imet 10.0.0.3
                                 10.0.0.3              -       100     0       65001 65004 i
 *  ec    RD: 10.0.0.3:30 imet 10.0.0.3
                                 10.0.0.3              -       100     0       65001 65004 i
Leaf1#

```
#### Маршруты на Spine1:
```
Spine1#show bgp evpn route-type mac-ip detail
BGP routing table information for VRF default
Router identifier 10.0.1.0, local AS number 65001
BGP routing table entry for mac-ip 0050.7966.6806, Route Distinguisher: 10.0.0.1:30
 Paths: 1 available
  65002
    10.0.0.1 from 10.2.1.1 (10.0.0.1)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:30:3000 TunnelEncap:tunnelTypeVxlan
      VNI: 3000 ESI: 0000:0000:0000:0000:0000
BGP routing table entry for mac-ip 0050.7966.6807, Route Distinguisher: 10.0.0.2:10
 Paths: 1 available
  65003
    10.0.0.2 from 10.2.1.3 (10.0.0.2)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:10:1000 TunnelEncap:tunnelTypeVxlan
      VNI: 1000 ESI: 0000:0000:0000:0000:0000
BGP routing table entry for mac-ip 0050.7966.6808, Route Distinguisher: 10.0.0.3:30
 Paths: 1 available
  65004
    10.0.0.3 from 10.2.1.5 (10.0.0.3)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:30:3000 TunnelEncap:tunnelTypeVxlan
      VNI: 3000 ESI: 0000:0000:0000:0000:0000
BGP routing table entry for mac-ip 0050.7966.6809, Route Distinguisher: 10.0.0.1:20
 Paths: 1 available
  65002
    10.0.0.1 from 10.2.1.1 (10.0.0.1)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:20:2000 TunnelEncap:tunnelTypeVxlan
      VNI: 2000 ESI: 0000:0000:0000:0000:0000
BGP routing table entry for mac-ip 0050.7966.680a, Route Distinguisher: 10.0.0.2:20
 Paths: 1 available
  65003
    10.0.0.2 from 10.2.1.3 (10.0.0.2)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:20:2000 TunnelEncap:tunnelTypeVxlan
      VNI: 2000 ESI: 0000:0000:0000:0000:0000
BGP routing table entry for mac-ip 0050.7966.680b, Route Distinguisher: 10.0.0.3:10
 Paths: 1 available
  65004
    10.0.0.3 from 10.2.1.5 (10.0.0.3)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:10:1000 TunnelEncap:tunnelTypeVxlan
      VNI: 1000 ESI: 0000:0000:0000:0000:0000
Spine1#
```
Route Target те, которые мы настроили, благодаря чему наши Leaf'ы смогут импортировать маршруты в нужный MAC-VRF.

#### Проверим наш EVI на примере Leaf1:
```
Leaf1#show bgp evpn instance
EVPN instance: VLAN 20
  Route distinguisher: 0:0
  Route target import: Route-Target-AS:20:2000
  Route target export: Route-Target-AS:20:2000
  Service interface: VLAN-based
  Local VXLAN IP address: 10.0.0.1
  VXLAN: enabled
  MPLS: disabled
EVPN instance: VLAN 30
  Route distinguisher: 0:0
  Route target import: Route-Target-AS:30:3000
  Route target export: Route-Target-AS:30:3000
  Service interface: VLAN-based
  Local VXLAN IP address: 10.0.0.1
  VXLAN: enabled
  MPLS: disabled
Leaf1#
```

#### Ещё несколько команд вывода для проверки 
```
Leaf1#show vxlan control-plane
   VLAN       Control Plane       Direction    Source
---------- ------------------- --------------- -------------
   20         EVPN                both         configuration
   30         EVPN                both         configuration
```
```
Leaf1#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP           Tunnel Type(s)
-------------- --------------
10.0.0.2       flood
10.0.0.3       flood

Total number of remote VTEPS:  2
Leaf1#
```

### Мы убедились, что клиенты видят друг друга на 2-ом уровне





