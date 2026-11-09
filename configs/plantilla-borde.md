# Plantilla del router de borde (NAT overload + eBGP)

Tarea T24. La aplica cada integrante en el router de borde de su AS (T25 a T28).

## Valores por AS

| AS | Responsable | IGP | Borde | Interfaces internas (inside) | Interfaz hacia R6 (outside) | IP del borde | IP de R6 |
|---|---|---|---|---|---|---|---|
| 1 | Emmanuel | OSPF | R4 | G0/0 (a R2), G0/1 (a R3) | G0/2 | 201.0.1.2/30 | 201.0.1.1 |
| 2 | Viviana | RIP | R2 | G0/0 (a R1), G0/1 (a R4) | G0/2 | 201.0.2.2/30 | 201.0.2.1 |
| 3 | Jerónimo | OSPF | R4 | G0/0 (a R2), G0/1 (a R3) | G0/2 | 201.0.3.2/30 | 201.0.3.1 |
| 4 | Simón | RIP | R2 | G0/0 (a R1), G0/1 (a R4) | G0/2 | 201.0.4.2/30 | 201.0.4.1 |

El router de borde debe ser un **Cisco 2911**, porque necesita un tercer puerto (G0/2) para R6.

## Bloque para AS con OSPF (AS1 y AS3)

Reemplaza `k` por el número del AS.

```
enable
configure terminal
interface g0/0
 ip nat inside
interface g0/1
 ip nat inside
interface g0/2
 description a-R6-AS99
 ip address 201.0.k.2 255.255.255.252
 ip nat outside
 no shutdown
exit
access-list 1 permit 10.0.0.0 0.0.0.255
access-list 1 permit 172.16.0.0 0.0.0.255
access-list 1 permit 192.168.0.0 0.0.255.255
ip nat inside source list 1 interface g0/2 overload
ip route 0.0.0.0 0.0.0.0 201.0.k.1
router bgp k
 neighbor 201.0.k.1 remote-as 99
exit
router ospf 1
 default-information originate
end
copy running-config startup-config
```

## Bloque para AS con RIP (AS2 y AS4)

Reemplaza `k` por el número del AS.

```
enable
configure terminal
interface g0/0
 ip nat inside
interface g0/1
 ip nat inside
interface g0/2
 description a-R6-AS99
 ip address 201.0.k.2 255.255.255.252
 ip nat outside
 no shutdown
exit
access-list 1 permit 10.0.0.0 0.0.0.255
access-list 1 permit 172.16.0.0 0.0.0.255
access-list 1 permit 192.168.0.0 0.0.255.255
ip nat inside source list 1 interface g0/2 overload
ip route 0.0.0.0 0.0.0.0 201.0.k.1
router bgp k
 neighbor 201.0.k.1 remote-as 99
exit
router rip
 default-information originate
end
copy running-config startup-config
```

La red 201.0.k.0/30 **no** se agrega al IGP del AS: es un enlace externo y su única salida es R6.

## Qué hace cada parte

| Parte | Función |
|---|---|
| `ip nat inside` | Marca las interfaces que miran hacia el interior del AS |
| `ip nat outside` | Marca la interfaz que mira hacia el AS99 |
| `access-list 1` | Define qué direcciones origen se traducen: las dos redes de clientes y las redes internas 192.168.x.x (enlaces y LAN del DHCP) |
| `ip nat inside source list 1 interface g0/2 overload` | Traduce todas esas direcciones a la única IP pública del borde (201.0.k.2), diferenciando cada conexión por puerto (PAT) |
| `router bgp k` + `neighbor` | Sesión eBGP con R6: el borde aprende las redes del AS99 (200.99.0.0/16) |
| `ip route 0.0.0.0 0.0.0.0 201.0.k.1` | Ruta por defecto del borde hacia R6: cubre los destinos que BGP no anuncia, como los enlaces 201.0.x.0/30 de los otros AS |
| `default-information originate` | El borde anuncia esa ruta por defecto dentro del AS, así los routers internos envían al borde todo lo que no conocen |

## Verificación

| Dónde | Comando | Resultado esperado |
|---|---|---|
| Borde | `show ip bgp summary` | Vecino 201.0.k.1 con un número en `State/PfxRcd` |
| Borde | `show ip route` | `B 200.99.0.0/16 via 201.0.k.1` y `S* 0.0.0.0/0 via 201.0.k.1` |
| Router interno | `show ip route` | `O*E2 0.0.0.0/0` (OSPF) o `R* 0.0.0.0/0` (RIP) hacia el borde |
| PC | `ping 200.99.1.10` | Responde (cuando el AS99 esté listo) |
| Borde | `show ip nat translations` | IP 10.0.0.x o 172.16.0.x traducida a 201.0.k.2 |

## Justificación para el informe

**Problema.** Los cuatro AS de clientes usan exactamente las mismas redes internas (10.0.0.0/24 y 172.16.0.0/24), y el enunciado prohíbe cambiarlas. Si esas redes se anunciaran hacia el AS99, R6 recibiría cuatro anuncios de 10.0.0.0/24 desde cuatro AS distintos. Solo instalaría uno, y las respuestas de los servidores llegarían siempre al mismo AS.

**Solución.** Cada router de borde aplica NAT overload (PAT). Todo el tráfico que sale del AS lleva como origen la IP pública del enlace con R6 (201.0.k.2), que es única para cada AS. El AS99 nunca ve las redes internas de los clientes: solo ve cuatro direcciones públicas distintas, todas conectadas directamente a R6, así que las respuestas siempre vuelven al AS correcto.

**Por qué PAT y no NAT estático.** Con PAT basta una sola IP pública por AS para todos los hosts, porque el router distingue las conexiones por número de puerto. Un NAT estático necesitaría una IP pública por host.

**Por qué eBGP.** Entre sistemas autónomos distintos el protocolo estándar es BGP. Cada borde forma una sesión eBGP con R6 y aprende de ella las redes de servicios (200.99.0.0/16). Los bordes no anuncian sus redes internas, porque están ocultas detrás del NAT.

**Por qué una ruta por defecto.** Los routers internos no necesitan conocer las redes del AS99: el borde es la única salida del AS. Por eso el borde tiene una ruta por defecto hacia R6 y la anuncia en el IGP, y así se evita redistribuir BGP dentro del AS. BGP sigue aportando la ruta específica a 200.99.0.0/16, que tiene prioridad sobre la ruta por defecto por ser más específica.

**Conectividad entre AS de clientes.** Un cliente del AS1 no puede alcanzar los hosts internos del AS2. Las direcciones son las mismas en ambos lados (un ping a 10.0.0.11 desde el AS1 se queda dentro del AS1), y el NAT solo traduce conexiones que se inician desde adentro, así que no existe forma de llegar a un host interno desde afuera. Lo único alcanzable entre clientes es la IP pública del borde de cada AS (201.0.k.2): el paquete sale por la ruta por defecto hacia R6, que tiene conectadas todas las redes 201.0.k.0/30, y lo responde el propio router de borde.
