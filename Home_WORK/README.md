
# Проектирование адресного пространства

### Цель:
- Собрать схему CLOS;
- Распределить адресное пространство;
- Выполнить настройки на стенде.
#### Соберём топологию как на схеме

![Схема](https://github.com/Dmitriy5588/OTUS/blob/main/Home_WORK/%D0%A2%D0%BE%D0%BF%D0%BE%D0%BB%D0%BE%D0%B3%D0%B8%D1%8F%20%D1%81%D0%B5%D1%82%D0%B8.png)

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

### Настройка стенда
#### Настройки на Spine1
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
interface Loopback1
   ip address 10.0.1.0/32
!
ip routing
!
end
Spine1#
```
#### Настройки на Spine2
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
interface Loopback1
   ip address 10.0.2.0/32
!
ip routing
!
end
Spine2#
```
#### Настройки на Leaf1
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
interface Loopback1
   ip address 10.0.0.1/32
!
ip routing
!
end
Leaf1#
```
#### Настройки на Leaf2
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
interface Loopback1
   ip address 10.0.0.2/32
!
ip routing
!
end
Leaf2#
```
#### Настройки на Leaf3
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
interface Loopback1
   ip address 10.0.0.3/32
!
ip routing
!
end
Leaf3#
```

### Проверка подключённых устройств по протоколу LLDP
#### Вводим команду *__show lldp neighbors__*
#### Вывод на Spine1
```
Spine1#show lldp neighbors
Last table change time   : 4:12:03 ago
Number of table inserts  : 6
Number of table deletes  : 3
Number of table drops    : 0
Number of table age-outs : 3

Port          Neighbor Device ID       Neighbor Port ID    TTL
---------- ------------------------ ---------------------- ---
Et1           Leaf1                    Ethernet1           120
Et2           Leaf2                    Ethernet1           120
Et3           Leaf3                    Ethernet1           120

Spine1#
```
#### Вывод на Spine2
```
Spine2#show lldp neighbors
Last table change time   : 4:12:57 ago
Number of table inserts  : 6
Number of table deletes  : 3
Number of table drops    : 0
Number of table age-outs : 3

Port          Neighbor Device ID       Neighbor Port ID    TTL
---------- ------------------------ ---------------------- ---
Et1           Leaf1                    Ethernet2           120
Et2           Leaf2                    Ethernet2           120
Et3           Leaf3                    Ethernet2           120

Spine2#
```

