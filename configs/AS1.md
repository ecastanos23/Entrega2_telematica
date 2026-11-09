# AS1 - Configuraciones

Responsable: Emmanuel Castaño Sepúlveda · IGP: OSPF (proceso 1, área 0) · Router de borde: R4 (AS 1 en BGP) · Packet Tracer 9.0

Cada bloque es la salida de `show running-config` (sin las líneas `!` vacías repetidas). Para aplicarlo en otro archivo: CLI del router → `enable` → `configure terminal` → pegar el bloque.

## Equipos

| Equipo | Modelo | Interfaces usadas |
|---|---|---|
| R1 | Cisco 2911 | G0/0 VLAN 10 (10.0.0.1/24), G0/1 a R2, G0/2 a R3 |
| R2 | Cisco 2911 | G0/0 a R1, G0/1 a R4 |
| R3 | PT-Router (módulos PT-ROUTER-NM-1CFE en slots 0, 1, 4 y 5) | Fa0/0 a R1, Fa1/0 a R4, Fa4/0 red 172.16.0.0/24, Fa5/0 LAN DHCP |
| R4 | Cisco 2911 | G0/0 a R2 (NAT inside), G0/1 a R3 (NAT inside), G0/2 a R6 (201.0.1.2/30, NAT outside) |
| S1 | Cisco 2960 | Fa0/1-24 y G0/1 en VLAN 10 |
| PC0, PC1 | PC-PT | S1 Fa0/1 y Fa0/2, IP por DHCP |
| PC2 | PC-PT | R3 Fa4/0, IP por DHCP |
| DHCP | Server-PT | R3 Fa5/0, IP fija 192.168.1.10 |

## Direccionamiento

| Enlace / red | Red | Direcciones |
|---|---|---|
| VLAN 10 | 10.0.0.0/24 | R1 .1, DHCP .11–.254 |
| Red 172 | 172.16.0.0/24 | R3 .1, DHCP .11–.254 |
| LAN DHCP | 192.168.1.0/24 | R3 .1, servidor .10 |
| R1–R2 | 192.168.0.0/30 | R1 .1, R2 .2 |
| R1–R3 | 192.168.0.4/30 | R1 .5, R3 .6 |
| R2–R4 | 192.168.0.8/30 | R2 .9, R4 .10 |
| R3–R4 | 192.168.0.12/30 | R3 .13, R4 .14 |
| R4–R6 (hacia AS99) | 201.0.1.0/30 | R6 .1, R4 .2 (IP pública del NAT) |

## R1

```
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname R1
!
ip cef
no ipv6 cef
!
license udi pid CISCO2911/K9 sn FTX1524LP97-
!
no ip domain-lookup
!
spanning-tree mode pvst
!
interface GigabitEthernet0/0
 description VLAN10
 ip address 10.0.0.1 255.255.255.0
 ip helper-address 192.168.1.10
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 description a-R2
 ip address 192.168.0.1 255.255.255.252
 duplex auto
 speed auto
!
interface GigabitEthernet0/2
 description a-R3
 ip address 192.168.0.5 255.255.255.252
 duplex auto
 speed auto
!
interface Vlan1
 no ip address
 shutdown
!
router ospf 1
 router-id 1.1.1.1
 log-adjacency-changes
 passive-interface GigabitEthernet0/0
 network 10.0.0.0 0.0.0.255 area 0
 network 192.168.0.0 0.0.0.3 area 0
 network 192.168.0.4 0.0.0.3 area 0
!
ip classless
!
ip flow-export version 9
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
end
```

## R2

```
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname R2
!
ip cef
no ipv6 cef
!
license udi pid CISCO2911/K9 sn FTX1524CXN6-
!
no ip domain-lookup
!
spanning-tree mode pvst
!
interface GigabitEthernet0/0
 description a-R1
 ip address 192.168.0.2 255.255.255.252
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 description a-R4
 ip address 192.168.0.9 255.255.255.252
 duplex auto
 speed auto
!
interface GigabitEthernet0/2
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface Vlan1
 no ip address
 shutdown
!
router ospf 1
 router-id 2.2.2.2
 log-adjacency-changes
 network 192.168.0.0 0.0.0.3 area 0
 network 192.168.0.8 0.0.0.3 area 0
!
ip classless
!
ip flow-export version 9
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
end
```

## R3

```
version 12.2
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname R3
!
no ip cef
no ipv6 cef
!
no ip domain-lookup
!
interface FastEthernet0/0
 description a-R1
 ip address 192.168.0.6 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet1/0
 description a-R4
 ip address 192.168.0.13 255.255.255.252
 duplex auto
 speed auto
!
interface Serial2/0
 no ip address
 clock rate 2000000
 shutdown
!
interface Serial3/0
 no ip address
 clock rate 2000000
 shutdown
!
interface FastEthernet4/0
 description RED-172
 ip address 172.16.0.1 255.255.255.0
 ip helper-address 192.168.1.10
 duplex auto
 speed auto
!
interface FastEthernet5/0
 description LAN-DHCP
 ip address 192.168.1.1 255.255.255.0
 duplex auto
 speed auto
!
router ospf 1
 router-id 3.3.3.3
 log-adjacency-changes
 passive-interface FastEthernet4/0
 passive-interface FastEthernet5/0
 network 192.168.0.4 0.0.0.3 area 0
 network 192.168.0.12 0.0.0.3 area 0
 network 172.16.0.0 0.0.0.255 area 0
 network 192.168.1.0 0.0.0.255 area 0
!
ip classless
!
ip flow-export version 9
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
end
```

## R4 (router de borde)

NAT overload, eBGP con R6 y ruta por defecto anunciada en OSPF, según `configs/plantilla-borde.md`.

```
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname R4
!
ip cef
no ipv6 cef
!
license udi pid CISCO2911/K9 sn FTX15240Z87-
!
no ip domain-lookup
!
spanning-tree mode pvst
!
interface GigabitEthernet0/0
 description a-R2
 ip address 192.168.0.10 255.255.255.252
 ip nat inside
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 description a-R3
 ip address 192.168.0.14 255.255.255.252
 ip nat inside
 duplex auto
 speed auto
!
interface GigabitEthernet0/2
 description a-R6-AS99
 ip address 201.0.1.2 255.255.255.252
 ip nat outside
 duplex auto
 speed auto
!
interface Vlan1
 no ip address
 shutdown
!
router ospf 1
 router-id 4.4.4.4
 log-adjacency-changes
 network 192.168.0.8 0.0.0.3 area 0
 network 192.168.0.12 0.0.0.3 area 0
 default-information originate
!
router bgp 1
 bgp log-neighbor-changes
 no synchronization
 neighbor 201.0.1.1 remote-as 99
!
ip nat inside source list 1 interface GigabitEthernet0/2 overload
ip classless
ip route 0.0.0.0 0.0.0.0 201.0.1.1
!
ip flow-export version 9
!
access-list 1 permit 10.0.0.0 0.0.0.255
access-list 1 permit 172.16.0.0 0.0.0.255
access-list 1 permit 192.168.0.0 0.0.255.255
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
end
```

## S1

```
version 15.0
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname S1
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface FastEthernet0/1
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/2
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/3
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/4
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/5
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/6
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/7
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/8
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/9
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/10
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/11
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/12
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/13
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/14
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/15
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/16
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/17
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/18
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/19
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/20
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/21
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/22
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/23
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/24
 switchport access vlan 10
 switchport mode access
!
interface GigabitEthernet0/1
 switchport access vlan 10
 switchport mode access
!
interface GigabitEthernet0/2
!
interface Vlan1
 no ip address
 shutdown
!
line con 0
!
line vty 0 4
 login
line vty 5 15
 login
!
end
```

## Servidor DHCP

Server-PT, se configura por interfaz gráfica.

- **Desktop → IP Configuration (Static):** IP 192.168.1.10, máscara 255.255.255.0, gateway 192.168.1.1, DNS 200.99.1.10
- **Services → DHCP:** Service On

| Pool | Gateway | DNS | IP inicial | Máscara | Usuarios |
|---|---|---|---|---|---|
| VLAN10 | 10.0.0.1 | 200.99.1.10 | 10.0.0.11 | 255.255.255.0 | 244 |
| RED172 | 172.16.0.1 | 200.99.1.10 | 172.16.0.11 | 255.255.255.0 | 244 |

El `serverPool` por defecto de Packet Tracer no se modifica.

## R6-temp (solo para pruebas, no va en el archivo maestro)

Router temporal en `AS1_Emmanuel_T25.pkt` que imita a R6 del AS99 para probar T25. Los loopbacks simulan las IP del DNS (200.99.1.10) y del WEB (200.99.2.10). En la integración (T29) lo reemplaza el R6 real.

```
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname R6-temp
!
ip cef
no ipv6 cef
!
license udi pid CISCO2911/K9 sn FTX15248P39-
!
no ip domain-lookup
!
spanning-tree mode pvst
!
interface Loopback1
 description simula-LAN-DNS
 ip address 200.99.1.10 255.255.255.0
!
interface Loopback2
 description simula-LAN-WEB
 ip address 200.99.2.10 255.255.255.0
!
interface GigabitEthernet0/0
 ip address 201.0.1.1 255.255.255.252
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface GigabitEthernet0/2
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface Vlan1
 no ip address
 shutdown
!
router bgp 99
 bgp log-neighbor-changes
 no synchronization
 neighbor 201.0.1.2 remote-as 1
 network 200.99.0.0 mask 255.255.0.0
!
ip classless
ip route 200.99.0.0 255.255.0.0 Null0
!
ip flow-export version 9
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
end
```
