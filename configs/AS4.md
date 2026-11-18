# AS4 - Configuración de Simón

Sistema autónomo: AS4 · IGP: RIP v2 · Router de borde: R2 · BGP AS: 4

Archivo de trabajo Packet Tracer: `packet-tracer/borradores/AS4_Simon.pkt` (todavía no existe; crea un archivo con **File > New**, arma la topología y guárdalo con **File > Save As**). Usa `AS2_viviana.pkt` como referencia visual, sin modificarlo. Esta configuración sigue la topología de AS2, cambiando el enlace externo a `201.0.4.0/30`. No cambies las redes internas, aunque se repitan en los otros AS.

## Equipos y cableado

| Conexión | Puerto 1 | Puerto 2 | Cable |
|---|---|---|---|
| R1–S1 | R1 G0/0 | S1 Fa0/1 | Copper Straight-Through |
| PC0–S1 | PC0 Fa0 | S1 Fa0/2 | Copper Straight-Through |
| PC1–S1 | PC1 Fa0 | S1 Fa0/3 | Copper Straight-Through |
| R1–R2 | R1 G0/1 | R2 G0/0 | Copper Cross-Over |
| R1–R3 | R1 G0/2 | R3 Fa4/0 | Copper Cross-Over |
| R2–R4 | R2 G0/1 | R4 G0/0 | Copper Cross-Over |
| R3–R4 | R3 Fa5/0 | R4 G0/1 | Copper Cross-Over |
| R3–PC2 | R3 Fa0/0 | PC2 Fa0 | Copper Cross-Over |
| R3–servidor DHCP | R3 Fa1/0 | DHCP Server Fa0 | Copper Cross-Over |
| R2–R6 (AS99) | R2 G0/2 | R6 Fa3/0 | Copper Cross-Over |

R1, R2 y R4: Cisco 2911. R3: PT-Router con puertos FastEthernet de cobre en Fa0/0, Fa1/0, Fa4/0 y Fa5/0. Si falta alguno, apaga R3, instala los módulos PT-ROUTER-NM-1CFE necesarios, enciéndelo y vuelve a pegar su configuración. R2 necesita G0/2 libre para R6.

## Direccionamiento

| Segmento | Interfaz/equipo | Dirección |
|---|---|---|
| VLAN 10 `10.0.0.0/24` | R1 G0/0 | `10.0.0.1/24` |
| LAN de PC2 `172.16.0.0/24` | R3 Fa0/0 | `172.16.0.1/24` |
| LAN DHCP `192.168.1.0/24` | R3 Fa1/0 / servidor | `192.168.1.1/24` / `192.168.1.10/24` |
| R1–R2 `192.168.0.0/30` | R1 G0/1 / R2 G0/0 | `.1` / `.2` |
| R1–R3 `192.168.0.4/30` | R1 G0/2 / R3 Fa4/0 | `.5` / `.6` |
| R2–R4 `192.168.0.8/30` | R2 G0/1 / R4 G0/0 | `.9` / `.10` |
| R3–R4 `192.168.0.12/30` | R3 Fa5/0 / R4 G0/1 | `.13` / `.14` |
| AS4–AS99 `201.0.4.0/30` | R2 G0/2 / R6 Fa3/0 | `.2` / `.1` |

## R1

```cisco
enable
configure terminal
hostname R1
interface GigabitEthernet0/0
 description VLAN10
 ip address 10.0.0.1 255.255.255.0
 ip helper-address 192.168.1.10
 no shutdown
exit
interface GigabitEthernet0/1
 description a-R2
 ip address 192.168.0.1 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/2
 description a-R3
 ip address 192.168.0.5 255.255.255.252
 no shutdown
exit
router rip
 version 2
 network 10.0.0.0
 network 192.168.0.0
 no auto-summary
end
copy running-config startup-config
```

## R2 - borde AS4, RIP, NAT/PAT y eBGP

```cisco
enable
configure terminal
hostname R2
interface GigabitEthernet0/0
 description a-R1
 ip address 192.168.0.2 255.255.255.252
 ip nat inside
 no shutdown
exit
interface GigabitEthernet0/1
 description a-R4
 ip address 192.168.0.9 255.255.255.252
 ip nat inside
 no shutdown
exit
interface GigabitEthernet0/2
 description a-R6-AS99
 ip address 201.0.4.2 255.255.255.252
 ip nat outside
 no shutdown
exit
access-list 1 permit 10.0.0.0 0.0.0.255
access-list 1 permit 172.16.0.0 0.0.0.255
access-list 1 permit 192.168.0.0 0.0.255.255
ip nat inside source list 1 interface GigabitEthernet0/2 overload
ip route 0.0.0.0 0.0.0.0 201.0.4.1
router bgp 4
 bgp log-neighbor-changes
 neighbor 201.0.4.1 remote-as 99
exit
router rip
 version 2
 network 192.168.0.0
 default-information originate
 no auto-summary
end
copy running-config startup-config
```

El vecino eBGP en R6 (`neighbor 201.0.4.2 remote-as 4`) le corresponde al responsable de AS99/integración en la tarea T29; coordina que esté configurado antes de probar BGP.

## R3 - LAN de PC2, servidor DHCP y relay

```cisco
enable
configure terminal
hostname R3
interface FastEthernet0/0
 description LAN_PC2
 ip address 172.16.0.1 255.255.255.0
 ip helper-address 192.168.1.10
 no shutdown
exit
interface FastEthernet1/0
 description LAN_DHCP
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit
interface FastEthernet4/0
 description a-R1
 ip address 192.168.0.6 255.255.255.252
 no shutdown
exit
interface FastEthernet5/0
 description a-R4
 ip address 192.168.0.13 255.255.255.252
 no shutdown
exit
router rip
 version 2
 network 172.16.0.0
 network 192.168.0.0
 network 192.168.1.0
 no auto-summary
end
copy running-config startup-config
```

## R4

```cisco
enable
configure terminal
hostname R4
interface GigabitEthernet0/0
 description a-R2
 ip address 192.168.0.10 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/1
 description a-R3
 ip address 192.168.0.14 255.255.255.252
 no shutdown
exit
router rip
 version 2
 network 192.168.0.0
 no auto-summary
end
copy running-config startup-config
```

## S1 - VLAN 10

Conecta el puerto Fa0/1 del switch a R1 G0/0; PC0 y PC1 a Fa0/2 y Fa0/3, como en la tabla de cableado.

```cisco
enable
configure terminal
hostname S1
vlan 10
 name CLIENTES
exit
interface range FastEthernet0/1 - 3
 switchport mode access
 switchport access vlan 10
 no shutdown
exit
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 10
 no shutdown
end
copy running-config startup-config
```

## Servidor DHCP (configuración en la interfaz gráfica)

En **Desktop > IP Configuration**, fija IP `192.168.1.10`, máscara `255.255.255.0`, gateway `192.168.1.1` y DNS `200.99.1.10`. En **Services > DHCP**, enciende DHCP y agrega (botón **Add**, no Save sobre `serverPool`) estos dos pools:

| Pool Name | Default Gateway | DNS Server | Start IP Address | Máscara | Máximo de usuarios |
|---|---|---|---|---|---:|
| `AS4_VLAN10` | `10.0.0.1` | `200.99.1.10` | `10.0.0.11` | `255.255.255.0` | 244 |
| `AS4_PC2` | `172.16.0.1` | `200.99.1.10` | `172.16.0.11` | `255.255.255.0` | 244 |

En cada PC, **Desktop > IP Configuration > DHCP**.

## Verificación y evidencias

Espera convergencia antes de probar. Ejecuta:

| Equipo | Comando/prueba | Resultado esperado |
|---|---|---|
| R1–R4 | `show ip interface brief` | Interfaces conectadas con IP y estado `up/up` |
| R1, R2, R3, R4 | `show ip route` | Rutas RIP marcadas `R` a las redes internas |
| R2 | `show ip route` | Tras T29, ruta `B` a `200.99.0.0/16` y `S*` por `201.0.4.1` |
| R2 | `show ip bgp summary` | Vecino `201.0.4.1` establecido; PfxRcd numérico |
| PC0, PC1, PC2 | `ipconfig /all` | IP del pool correcto, gateway `.1` de su red y DNS `200.99.1.10` |
| PC0 → PC2 | `ping 172.16.0.11` | Responde dentro de AS4 |
| PC2 → DHCP | `ping 192.168.1.10` | Responde |
| PC de AS4 → DNS/WEB | ping `200.99.1.10`, `200.99.2.10` | Responden cuando AS99 y T29 estén listos |

Guarda capturas en `evidencias/AS4/`: `PC0_ipconfig.png`, `PC2_ipconfig.png`, `PC0_ping_PC2.png`, `R2_show_ip_route.png`, `R2_show_ip_bgp_summary.png`, `R2_show_ip_nat_translations.png` y las pruebas requeridas por T20. Guarda los routers con `copy running-config startup-config`.
