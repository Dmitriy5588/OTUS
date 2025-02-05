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




