# AS3 - Configuraciones

Responsable: Jerónimo Contreras Sierra · IGP: OSPF (proceso 1, área 0) · Router de borde: R4 (AS 3 en BGP) · Packet Tracer 9.0.1

Cada bloque es la salida de `show running-config` (sin las líneas `!` vacías repetidas). Para aplicarlo en otro archivo: CLI del router → `enable` → `configure terminal` → pegar el bloque. Archivo de trabajo: `packet-tracer/borradores/AS3_AS99_Jeronimo.pkt`. El AS3 tiene la misma estructura que el AS1; solo cambia el enlace hacia el AS99 (201.0.3.0/30).

## Equipos

| Equipo | Modelo | Interfaces usadas |
|---|---|---|
| R1 | Cisco 2911 | G0/0 VLAN 10 (10.0.0.1/24), G0/1 a R2, G0/2 a R3 |
| R2 | Cisco 2911 | G0/0 a R1, G0/1 a R4 |
| R3 | PT-Router (módulos PT-ROUTER-NM-1CFE en slots 0, 1, 4 y 5) | Fa0/0 a R1, Fa1/0 a R4, Fa4/0 red 172.16.0.0/24, Fa5/0 LAN DHCP |
| R4 | Cisco 2911 (nombre en el lienzo: R4-AS3) | G0/0 a R2 (NAT inside), G0/1 a R3 (NAT inside), G0/2 a R6 Fa2/0 (201.0.3.2/30, NAT outside) |
| S1 | Cisco 2960 | Fa0/1-24 y G0/1 en VLAN 10; G0/1 a R1 G0/0 |
| PC0, PC1 | PC-PT | S1 Fa0/1 y Fa0/2, IP por DHCP |
| PC2 | PC-PT | R3 Fa4/0, IP por DHCP |
| DHCP | Server-PT | R3 Fa5/0, IP fija 192.168.1.10 |

Cables: Copper Straight-Through entre S1 y R1, PC0 y PC1 (un extremo es switch); Copper Cross-Over en los demás enlaces (router-router, router-PC y router-servidor).

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
| R4–R6 (hacia AS99) | 201.0.3.0/30 | R6 .1, R4 .2 (IP pública del NAT) |

## R1

```
!
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
license udi pid CISCO2911/K9 sn FTX1524BTQ1-
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
!
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
license udi pid CISCO2911/K9 sn FTX1524KJRW-
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
!
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
 shutdown
!
interface Serial3/0
 no ip address
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

En el borrador, R6 ya tiene `neighbor 201.0.3.2 remote-as 3` para probar la sesión. En el maestro, ese vecino lo agrega Emmanuel en T29.

```
!
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
license udi pid CISCO2911/K9 sn FTX1524LK6D-
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
 ip address 201.0.3.2 255.255.255.252
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
router bgp 3
 bgp log-neighbor-changes
 no synchronization
 neighbor 201.0.3.1 remote-as 99
!
ip nat inside source list 1 interface GigabitEthernet0/2 overload
ip classless
ip route 0.0.0.0 0.0.0.0 201.0.3.1
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
!
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

## Verificación

Capturas en `evidencias/AS3/`.

| Dónde | Comando | Resultado obtenido | Captura |
|---|---|---|---|
| S1 | `show vlan brief` | VLAN 10 activa con Fa0/1-24 y Gig0/1 | `S1_show_vlan.png` |
| PC0, PC1, PC2 | `ipconfig` | 10.0.0.11, 10.0.0.12 y 172.16.0.11, recibidas por DHCP a través del relay | `PC0_ipconfig.png`, `PC1_ipconfig.png`, `PC2_ipconfig.png` |
| DHCP | Services → DHCP | Pools VLAN10 y RED172 | `DHCP_pools.png` |
| PC0 | `ping` / `tracert` a PC2 y al DHCP | Responden | `PC0_ping_PC2_DHCP_tracert.png` |
| R1 | `show ip ospf neighbor`, `show ip route` | 2.2.2.2 y 3.3.3.3 FULL; ruta `O*E2 0.0.0.0/0` | `R1_ospf_neighbor_y_route.png` |
| R4 | `show ip ospf neighbor`, `ping 192.168.1.10` | 2.2.2.2 y 3.3.3.3 FULL; ping al DHCP 100 % | `R4_ospf_neighbor_y_ping_DHCP.png` |
| R4 | `show ip bgp summary`, `show ip route` | Vecino 201.0.3.1 (AS 99) establecido; `B 200.99.0.0/16 [20/0] via 201.0.3.1` | `R4_bgp_summary_y_route.png` |
| R6 | `show ip bgp` | `*> 200.99.0.0/16` anunciado al AS3 | `../AS99/R6_show_ip_bgp.png` |
| PC0 | `ping 200.99.1.10`, `ping 200.99.2.10` | Responden el DNS y el WEB (TTL=123) | `PC0_ping_DNS_WEB.png` |
| PC2 | `ping 200.99.1.10` | Responde el DNS (TTL=124) | `PC2_ping_DNS.png` |
| R4 | `show ip nat translations` | 10.0.0.11 y 172.16.0.11 traducidas a 201.0.3.2 | `R4_nat_translations.png` |
| Topología | — | Todos los enlaces del AS3 y del AS99 en verde | `AS3_topologia.png` |

## Justificación para el informe

**OSPF dentro del AS3.** El AS3 tiene cuatro routers en anillo (R1–R2–R4–R3–R1), así que hay dos caminos entre la LAN de usuarios y el borde. OSPF elige el de menor costo y reconverge si cae un enlace. Se usa OSPF en AS1, AS3 y AS99, y RIP en AS2 y AS4, para no usar un solo IGP en toda la topología.

**DHCP centralizado con relay.** Hay un solo servidor DHCP (192.168.1.10) para dos redes de usuarios. Como el DHCP usa broadcast y los routers no lo reenvían, R1 (G0/0) y R3 (Fa4/0) tienen `ip helper-address 192.168.1.10`: convierten el broadcast en unicast hacia el servidor, que elige el pool según la red de la interfaz que lo reenvió.

**passive-interface en las LAN.** Las redes 10.0.0.0/24, 172.16.0.0/24 y 192.168.1.0/24 solo tienen hosts. Se anuncian en OSPF, pero no se envían hellos hacia ellas.

**Ruta por defecto y default-information originate.** R4 es la única salida del AS3. Tiene `ip route 0.0.0.0 0.0.0.0 201.0.3.1` y la reparte en OSPF con `default-information originate`; los demás routers la ven como `O*E2`. Así no hace falta redistribuir BGP dentro del AS3.

**NAT overload (PAT) en R4.** Las redes internas son privadas y no pueden salir al AS99 tal cual. R4 traduce todo lo que permite `access-list 1` a su IP pública 201.0.3.2 (inside en G0/0 y G0/1, outside en G0/2). Como el AS99 ve una sola dirección por cliente, basta con que conozca la red 201.0.3.0/30.

**eBGP con el AS99.** R4 forma la sesión eBGP con R6 (201.0.3.1, AS 99) y aprende el resumen `200.99.0.0/16`, donde están el DNS y el WEB. El AS3 no anuncia redes internas por BGP: son privadas y quedan ocultas detrás del NAT.
