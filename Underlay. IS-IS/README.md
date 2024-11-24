# Построение сети Underlay на основе протокола IS-IS.
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
где 49 - указывает тип адреса, 0100 - номер зоны, ХХХХ.ХХХХ.ХХХХ - ID устройства, в качестве System ID будем преобразовывать ip адрес Loopback1, 00 - селектор (всегда ноль).
```
router isis 100
   net 49.0100.0100.0000.1000.00
   !
   address-family ipv4 unicast
```
Команда __address-family__ включает семейства адресов, которые IS-IS будет маршрутизировать, и переводит коммутатор в режим конфигурации для этого семейства адресов.

__Настройки на интерфейсах.__
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




