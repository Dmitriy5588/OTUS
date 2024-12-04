# Построение сети Underlay на основе протокола eBGP
### Цель:
- Настроить eBGP для Underlay сети;
- Проверить IP связность между сетевыми устройствами.
### Краткое описание протокола
__BGP (Border Gateway Protocol)__ — это основной протокол динамической маршрутизации, который используется в Интернете.

Лучший маршрут выбирается с помощью специальных атрибутов. Один из основных атрибутов, который передается с информацией о маршруте — это список автономных систем, через которые прошла эта информация.

Существует два протокала BGP:
- Внутренний BGP (Internal BGP, iBGP) — BGP работающий внутри автономной системы. iBGP-соседи не обязательно должны быть непосредственно соединены.
- Внешний BGP (External BGP, eBGP) — BGP работающий между автономными системами. По умолчанию, eBGP-соседи должны быть непосредственно соединены.

BGP это __path-vector__ протокол с такими общими характеристиками:
- Использует TCP для передачи данных, это обеспечивает надежную доставку обновлений протокола (порт 179)
- Отправляет обновления только после изменений в сети (нет периодических обновлений)
- Периодически отправляет keepalive-сообщения для проверки TCP-соединения
- Метрика протокола называется path vector или атрибуты (attributes)

Примеры атрибутов BGP:
- Autonomous system path
- Next-hop
- Origin
- Local preference
- Atomic aggregate

### Описание сети
Топология сети и адреса сетевых устройств были ранее сконфигурированны на лабораторном стенде.
Сетевые устройства будут поделены на автномные системы (autonomous system, AS). Два устройства Spine, принадлежат одной AS65001, каждое устройства Leaf принадлежит своей AS (AS65002, AS65003, AS65004).

Адреса интерфесов сетевых устройств настроены с маской __/31__ для peer линков.
### Схема сети
![](https://github.com/Dmitriy5588/OTUS/blob/main/Underlay.%20BGP/eBGP.png)
#### Таблица адресов

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

### Настройка сетевых устройств
#### Настройка BGP на Spine1 и Spine2
Запускаем процес BGP, указываем router-id, статично указываем соседей.
```
!
router bgp 65001
   router-id 10.0.1.0
   neighbor 10.2.1.1 remote-as 65002
   neighbor 10.2.1.1 bfd
   neighbor 10.2.1.3 remote-as 65003
   neighbor 10.2.1.3 bfd
   neighbor 10.2.1.5 remote-as 65004
   neighbor 10.2.1.5 bfd
   network 10.0.1.0/32
   redistribute connected route-map Connected_Network
!
end
Spine1#
```
Анонсируем адрес Loopback1 командой *__network__*.

Для более гибкого обслуживание существующих сетей и возможных дополнительных подключений создадим *__route-map__* и *__ACL__* с нужными нам сетями для анонса в BGP.
```
!
route-map Connected_Network permit 10
   match ip address access-list Networks
!
```
```
!
ip access-list Networks
   10 permit ip 10.2.1.0/31 any
   20 permit ip 10.2.1.2/31 any
   30 permit ip 10.2.1.4/31 any
!
```
Итоговая конфигурация на Spine1 и Soine2:
<details>
  
<summary> Настройки на Spine1 </summary>

```
!
interface Ethernet1
   description to_Leaf1
   no switchport
   ip address 10.2.1.0/31
!
interface Ethernet2
   description to_Leaf2
   no switchport
   ip address 10.2.1.2/31
!
interface Ethernet3
   description to_Leaf3
   no switchport
   ip address 10.2.1.4/31
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
!
interface Loopback1
   ip address 10.0.1.0/32
!
interface Management1
!
ip access-list Networks
   10 permit ip 10.2.1.0/31 any
   20 permit ip 10.2.1.2/31 any
   30 permit ip 10.2.1.4/31 any
!
ip routing
!
route-map Connected_Network permit 10
   match ip address access-list Networks
!
router bgp 65001
   router-id 10.0.1.0
   neighbor 10.2.1.1 remote-as 65002
   neighbor 10.2.1.1 bfd
   neighbor 10.2.1.3 remote-as 65003
   neighbor 10.2.1.3 bfd
   neighbor 10.2.1.5 remote-as 65004
   neighbor 10.2.1.5 bfd
   network 10.0.1.0/32
   redistribute connected route-map Connected_Network
!
end
Spine1#
```
</details>

<details>
  
<summary> Настройки на Spine2 </summary>

```
!
interface Ethernet1
   description to_Leaf1
   no switchport
   ip address 10.2.2.0/31
!
interface Ethernet2
   description to_Leaf2
   no switchport
   ip address 10.2.2.2/31
!
interface Ethernet3
   description to_Leaf3
   no switchport
   ip address 10.2.2.4/31
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
!
interface Loopback1
   ip address 10.0.2.0/32
!
interface Management1
!
ip access-list Networks
   10 permit ip 10.2.2.0/31 any
   20 permit ip 10.2.2.2/31 any
   30 permit ip 10.2.2.4/31 any
!
ip routing
!
route-map Connected_Network permit 10
   match ip address access-list Networks
!
router bgp 65001
   router-id 10.0.2.0
   neighbor 10.2.2.1 remote-as 65002
   neighbor 10.2.2.1 bfd
   neighbor 10.2.2.3 remote-as 65003
   neighbor 10.2.2.3 bfd
   neighbor 10.2.2.5 remote-as 65004
   neighbor 10.2.2.5 bfd
   network 10.0.2.0/32
   redistribute connected route-map Connected_Networks
!
end
Spine2#
```
</details>

Настройки на Leaf-ах, за исключением настройки *__route-map__* и *__ACL__*.

<details>
  
<summary> Настройки на Leaf1 </summary>

```
!
interface Ethernet1
   description to_Spine1
   no switchport
   ip address 10.2.1.1/31
!
interface Ethernet2
   description to_Spine2
   no switchport
   ip address 10.2.2.1/31
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
!
interface Loopback1
   ip address 10.0.0.1/32
!
interface Management1
!
ip routing
!
router bgp 65002
   router-id 10.0.0.1
   neighbor 10.2.1.0 remote-as 65001
   neighbor 10.2.1.0 bfd
   neighbor 10.2.2.0 remote-as 65001
   neighbor 10.2.2.0 bfd
   network 10.0.0.1/32
   network 10.2.1.0/31
   network 10.2.2.0/31
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
!
interface Ethernet2
   description to_Spine2
   no switchport
   ip address 10.2.2.3/31
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
!
interface Loopback1
   ip address 10.0.0.2/32
!
interface Management1
!
ip routing
!
router bgp 65003
   router-id 10.0.0.2
   neighbor 10.2.1.2 remote-as 65001
   neighbor 10.2.1.2 bfd
   neighbor 10.2.2.2 remote-as 65001
   neighbor 10.2.2.2 bfd
   network 10.0.0.2/32
   network 10.2.1.2/31
   network 10.2.2.2/31
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
!
interface Ethernet2
   description to_Spine2
   no switchport
   ip address 10.2.2.5/31
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
!
interface Loopback1
   ip address 10.0.0.3/32
!
interface Management1
!
ip routing
!
router bgp 65004
   router-id 10.0.0.3
   neighbor 10.2.1.4 remote-as 65001
   neighbor 10.2.1.4 bfd
   neighbor 10.2.2.4 remote-as 65001
   neighbor 10.2.2.4 bfd
   network 10.0.0.3/32
   network 10.2.1.4/31
   network 10.2.2.4/31
!
end
Leaf3#
```
</details>

### Проверка установления соседства с устройствами в сети
Выполним команду __*show ip bgp summary*__ на устройстве Spine1.
```
Spine1#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.0.1.0, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.1         4  65002            138       274    0    0 02:45:43 Estab   3      3
  10.2.1.3         4  65003            143       281    0    0 02:45:40 Estab   3      3
  10.2.1.5         4  65004            137       274    0    0 02:45:43 Estab   3      3
Spine1#
```
### Проверка IP связности устройств
Для проверки выполним команду __*ping*__ c устройства Leaf1 до Leaf3. В качестве source укажем loopback1:
```
Leaf1#ping 10.0.0.3 source loopback 1
PING 10.0.0.3 (10.0.0.3) from 10.0.0.1 : 72(100) bytes of data.
80 bytes from 10.0.0.3: icmp_seq=1 ttl=63 time=9.09 ms
80 bytes from 10.0.0.3: icmp_seq=2 ttl=63 time=4.68 ms
80 bytes from 10.0.0.3: icmp_seq=3 ttl=63 time=4.83 ms
80 bytes from 10.0.0.3: icmp_seq=4 ttl=63 time=4.47 ms
80 bytes from 10.0.0.3: icmp_seq=5 ttl=63 time=4.22 ms

--- 10.0.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 35ms
rtt min/avg/max/mdev = 4.227/5.464/9.099/1.829 ms, ipg/ewma 8.758/7.206 ms
Leaf1#
```
IP связность с устройством Leaf3 есть.

Для проверки таблицы маршрутизации BGP выборочно выполним команду __*show ip route bgp*__ на устройстве Leaf3:
```
Leaf3#show ip route bgp

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

 B E      10.0.0.1/32 [200/0] via 10.2.1.4, Ethernet1
 B E      10.0.0.2/32 [200/0] via 10.2.1.4, Ethernet1
 B E      10.0.1.0/32 [200/0] via 10.2.1.4, Ethernet1
 B E      10.0.2.0/32 [200/0] via 10.2.2.4, Ethernet2
 B E      10.2.1.0/31 [200/0] via 10.2.2.4, Ethernet2
 B E      10.2.1.2/31 [200/0] via 10.2.2.4, Ethernet2
 B E      10.2.2.0/31 [200/0] via 10.2.2.4, Ethernet2
 B E      10.2.2.2/31 [200/0] via 10.2.2.4, Ethernet2

Leaf3#
```
Маршруты получены.
