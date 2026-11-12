# AS2 – Configuraciones

**Responsable:** Viviana Arango Tabares
**Sistema autónomo:** AS2  
**Protocolo de enrutamiento interno:** RIP versión 2  
**Router de borde:** R2  

---

## Equipos y topología

| Equipo | Función |
|---|---|
| R1 | Router de acceso a VLAN10 y conexión con R2 y R3 |
| R2 | Router de borde de AS2 y conexión con AS99 |
| R3 | Router de acceso a PC2 y al servidor DHCP |
| R4 | Router de interconexión entre R2 y R3 |
| S1 | Switch de acceso para VLAN10 |
| PC0 y PC1 | Clientes de VLAN10 |
| PC2 | Cliente de la red 172.16.0.0/24 |
| DHCP Server | Servidor DHCP de AS2 |

### Conexiones

| Dispositivo 1 | Interfaz A | Dispositivo 2 | Interfaz B |
|---|---|---|---|
| R1 | G0/0 | S1 | Fa0/1 |
| PC0 | Fa0 | S1 | Fa0/2 |
| PC1 | Fa0 | S1 | Fa0/3 |
| R3 | Fa0/0 | PC2 | Fa0 |
| R3 | Fa1/0 | DHCP Server | Fa0 |
| R1 | G0/1 | R2 | G0/0 |
| R1 | G0/2 | R3 | Fa4/0 |
| R2 | G0/1 | R4 | G0/0 |
| R3 | Fa5/0 | R4 | G0/1 |
| R2 | G0/2 | R6 | Fa1/0 |

---

## Direccionamiento

| Enlace | Red | Interfaz / dispositivo | Dirección IP |
|---|---|---|---|
| VLAN10 | 10.0.0.0/24 | R1 G0/0 | 10.0.0.1 |
| VLAN10 | 10.0.0.0/24 | PC0 | DHCP |
| VLAN10 | 10.0.0.0/24 | PC1 | DHCP |
| LAN PC2 | 172.16.0.0/24 | R3 Fa0/0 | 172.16.0.1 |
| LAN DHCP | 192.168.1.0/24 | R3 Fa1/0 | 192.168.1.1 |
| LAN DHCP | 192.168.1.0/24 | DHCP Server | 192.168.1.10 |
| R1 – R2 | 192.168.0.0/30 | R1 G0/1 | 192.168.0.1 |
| R1 – R2 | 192.168.0.0/30 | R2 G0/0 | 192.168.0.2 |
| R1 – R3 | 192.168.0.4/30 | R1 G0/2 | 192.168.0.5 |
| R1 – R3 | 192.168.0.4/30 | R3 Fa4/0 | 192.168.0.6 |
| R2 – R4 | 192.168.0.8/30 | R2 G0/1 | 192.168.0.9 |
| R2 – R4 | 192.168.0.8/30 | R4 G0/0 | 192.168.0.10 |
| R3 – R4 | 192.168.0.12/30 | R3 Fa5/0 | 192.168.0.13 |
| R3 – R4 | 192.168.0.12/30 | R4 G0/1 | 192.168.0.14 |
| AS2 – AS99 | 201.0.2.0/30 | R2 G0/2 | 201.0.2.2 |
| AS2 – AS99 | 201.0.2.0/30 | R6 Fa1/0 | 201.0.2.1 |

---

Cada bloque es la salida de `show running-config` (sin las líneas `!` vacías repetidas).

## Configuración de R1

```cisco
Building configuration...
Current configuration : 907 bytes
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
license udi pid CISCO2911/K9 sn FTX1524WHO1-
!
spanning-tree mode pvst
!
interface GigabitEthernet0/0
 description LAN_VLAN10
 ip address 10.0.0.1 255.255.255.0
 ip helper-address 192.168.1.10
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 description LINK_R2
 ip address 192.168.0.1 255.255.255.252
 duplex auto
 speed auto
!
interface GigabitEthernet0/2
 description LINK_R3
 ip address 192.168.0.5 255.255.255.252
 duplex auto
 speed auto
!
interface Vlan1
 no ip address
 shutdown
!
router rip
 version 2
 network 10.0.0.0
 network 192.168.0.0
 no auto-summary
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

---

## Configuración de R2

R2 funciona como router de borde de AS2. Tiene dos conexiones internas y una conexión hacia AS99.

```cisco
Building configuration...

Current configuration : 1222 bytes
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
license udi pid CISCO2911/K9 sn FTX1524V216-
!
spanning-tree mode pvst
!
interface GigabitEthernet0/0
 description LINK_R1
 ip address 192.168.0.2 255.255.255.252
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 description LINK_R4
 ip address 192.168.0.9 255.255.255.252
 duplex auto
 speed auto
!
interface GigabitEthernet0/2
 description LINK_AS99_R6
 ip address 201.0.2.2 255.255.255.252
 duplex auto
 speed auto
!
interface Vlan1
 no ip address
 shutdown
!
router rip
 version 2
 network 192.168.0.0
 no auto-summary
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
---

## Configuración de R3

R3 conecta la red de PC2, el servidor DHCP y los routers R1 y R4.

```cisco
Building configuration...

Current configuration : 1055 bytes
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
interface FastEthernet0/0
 description LAN_PC2
 ip address 172.16.0.1 255.255.255.0
 ip helper-address 192.168.1.10
 duplex auto
 speed auto
!
interface FastEthernet1/0
 description LAN_DHCP
 ip address 192.168.1.1 255.255.255.0
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
 description LINK_R1
 ip address 192.168.0.6 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet5/0
 description LINK_R4
 ip address 192.168.0.13 255.255.255.252
 duplex auto
 speed auto
!
router rip
 version 2
 network 172.16.0.0
 network 192.168.0.0
 network 192.168.1.0
 no auto-summary
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
---

## Configuración de R4

R4 permite la comunicación entre R2 y R3.

```cisco
Building configuration...

Current configuration : 825 bytes
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
license udi pid CISCO2911/K9 sn FTX1524613G-
!
spanning-tree mode pvst
!
interface GigabitEthernet0/0
 description LINK_R2
 ip address 192.168.0.10 255.255.255.252
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 description LINK_R3
 ip address 192.168.0.14 255.255.255.252
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
router rip
 version 2
 network 192.168.0.0
 no auto-summary
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
---

## Configuración de S1

Se crea la VLAN10 y se asignan los puertos utilizados por los equipos de la red.

```cisco
Building configuration...

Current configuration : 1233 bytes
!
version 15.0
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname Switch
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
!
interface FastEthernet0/5
!
interface FastEthernet0/6
!
interface FastEthernet0/7
!
interface FastEthernet0/8
!
interface FastEthernet0/9
!
interface FastEthernet0/10
!
interface FastEthernet0/11
!
interface FastEthernet0/12
!
interface FastEthernet0/13
!
interface FastEthernet0/14
!
interface FastEthernet0/15
!
interface FastEthernet0/16
!
interface FastEthernet0/17
!
interface FastEthernet0/18
!
interface FastEthernet0/19
!
interface FastEthernet0/20
!
interface FastEthernet0/21
!
interface FastEthernet0/22
!
interface FastEthernet0/23
!
interface FastEthernet0/24
!
interface GigabitEthernet0/1
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
---
Cada bloque es la salida de `ipconfig /all`

## Configuración de PC0

```cisco
FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 00D0.BA98.5B32
   Link-local IPv6 Address.........: FE80::2D0:BAFF:FE98:5B32
   IPv6 Address....................: ::
   IPv4 Address....................: 10.0.0.11
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     10.0.0.1
   DHCP Servers....................: 192.168.1.10
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-BD-40-E8-15-00-D0-BA-98-5B-32
   DNS Servers.....................: ::
                                     200.99.1.10

Bluetooth Connection:

   Connection-specific DNS Suffix..: 
   Physical Address................: 0001.436B.D194
   Link-local IPv6 Address.........: ::
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-BD-40-E8-15-00-D0-BA-98-5B-32
   DNS Servers.....................: ::
                                     200.99.1.10
```

## Configuración de PC1

```cisco
FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0001.96BB.6E95
   Link-local IPv6 Address.........: FE80::201:96FF:FEBB:6E95
   IPv6 Address....................: ::
   IPv4 Address....................: 10.0.0.12
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     10.0.0.1
   DHCP Servers....................: 192.168.1.10
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-D3-96-34-24-00-01-96-BB-6E-95
   DNS Servers.....................: ::
                                     200.99.1.10

Bluetooth Connection:

   Connection-specific DNS Suffix..: 
   Physical Address................: 00E0.B0AC.E33D
   Link-local IPv6 Address.........: ::
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-D3-96-34-24-00-01-96-BB-6E-95
   DNS Servers.....................: ::
                                     200.99.1.10
```

## Configuración de PC2

```cisco
FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 00E0.8FE6.7D4B
   Link-local IPv6 Address.........: FE80::2E0:8FFF:FEE6:7D4B
   IPv6 Address....................: ::
   IPv4 Address....................: 172.16.0.11
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     172.16.0.1
   DHCP Servers....................: 192.168.1.10
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-06-76-76-4E-00-E0-8F-E6-7D-4B
   DNS Servers.....................: ::
                                     200.99.1.10

Bluetooth Connection:

   Connection-specific DNS Suffix..: 
   Physical Address................: 0060.5C4B.EC52
   Link-local IPv6 Address.........: ::
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-06-76-76-4E-00-E0-8F-E6-7D-4B
   DNS Servers.....................: ::
                                     200.99.1.10
```
---

## Configuración del servidor DHCP

El servidor DHCP tiene una dirección IP estática y dos pools para asignar direcciones a los clientes de AS2.

### Configuración IP estática del servidor

| Parámetro | Valor |
|---|---|
| Dirección IP | 192.168.1.10 |
| Máscara | 255.255.255.0 |
| Gateway | 192.168.1.1 |
| DNS | 200.99.1.10 |

### Pool AS2_VLAN10

| Parámetro | Valor |
|---|---|
| Pool Name | AS2_VLAN10 |
| Default Gateway | 10.0.0.1 |
| DNS Server | 200.99.1.10 |
| Start IP Address | 10.0.0.11 |
| Subnet Mask | 255.255.255.0 |
| Maximum Number of Users | 244 |

### Pool AS2_PC2

| Parámetro | Valor |
|---|---|
| Pool Name | AS2_PC2 |
| Default Gateway | 172.16.0.1 |
| DNS Server | 200.99.1.10 |
| Start IP Address | 172.16.0.11 |
| Subnet Mask | 255.255.255.0 |
| Maximum Number of Users | 244 |

### DHCP Relay

El servidor recibe las solicitudes DHCP de las dos redes mediante las siguientes interfaces:

- R1 G0/0: `ip helper-address 192.168.1.10`
- R3 Fa0/0: `ip helper-address 192.168.1.10`