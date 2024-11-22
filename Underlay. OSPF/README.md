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

*__Аналогично выполняем настройки на всех сетевых устройствах в нашей AREA__*
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

