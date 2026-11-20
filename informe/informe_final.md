# Informe final – Interconexión de sistemas autónomos y acceso a servicios

Entrega 2 · Internet, Arquitectura y Protocolos (Telemática) · EAFIT · 2026-2

## Topología y direccionamiento

La implementación conecta cuatro sistemas autónomos de clientes (AS1–AS4) con el AS99, que ofrece los servicios DNS y WEB. Cada AS cliente tiene cuatro routers, un switch que conecta los equipos de la VLAN 10, tres PCs y un servidor DHCP. Los PCs de la segunda red de usuarios se conectan directamente a una interfaz de capa 3 de R3. El AS99 está compuesto por R5, R6 y R7; R6 conecta los cuatro AS clientes y R5 y R7 dan acceso a los servidores DNS y WEB.

La figura presenta la topología integrada de los cinco sistemas autónomos:

![Topología completa de los cinco sistemas autónomos](../evidencias/extremo-a-extremo/topologia_maestro.png)

| Sistema autónomo | Función | IGP |
|---|---|---|
| AS1 | Cliente | OSPF (proceso 1, área 0) |
| AS2 | Cliente | RIP versión 2 |
| AS3 | Cliente | OSPF (proceso 1, área 0) |
| AS4 | Cliente | RIP versión 2 |
| AS99 | Proveedor DNS/WEB | OSPF (proceso 1, área 0) |

Las redes internas de los AS de clientes se repiten según el direccionamiento definido para el proyecto. Los enlaces de tránsito entre routers usan subredes /30. Los PCs de usuario reciben direcciones por DHCP; las tablas indican sus redes y gateways, no direcciones individuales de concesiones salvo que estén documentadas en las configuraciones.

### AS1

| Segmento | Red | Interfaces y direcciones del maestro |
|---|---|---|
| VLAN 10 | `10.0.0.0/24` | R1 G0/0: `10.0.0.1`; PCs DHCP: `.11–.254` |
| Red de usuarios 2, conectada a R3 | `172.16.0.0/24` | R3 Fa4/0: `172.16.0.1`; PCs DHCP: `.11–.254` |
| LAN DHCP | `192.168.1.0/24` | R3 Fa5/0: `192.168.1.1`; servidor: `192.168.1.10` |
| R1–R2 | `192.168.0.0/30` | R1 G0/1: `.1`; R2 G0/0: `.2` |
| R1–R3 | `192.168.0.4/30` | R1 G0/2: `.5`; R3 Fa0/0: `.6` |
| R2–R4 | `192.168.0.8/30` | R2 G0/1: `.9`; R4 G0/0: `.10` |
| R3–R4 | `192.168.0.12/30` | R3 Fa1/0: `.13`; R4 G0/1: `.14` |
| AS1–R6 | `201.0.1.0/30` | R6 Fa0/0: `201.0.1.1`; R4 G0/2: `201.0.1.2` (IP de PAT) |

### AS2

| Segmento | Red | Interfaces y direcciones del maestro |
|---|---|---|
| VLAN 10 | `10.0.0.0/24` | R1 G0/0: `10.0.0.1`; PCs DHCP: `.11–.254` |
| Red de usuarios 2, conectada a R3 | `172.16.0.0/24` | R3 Fa4/0: `172.16.0.1`; PCs DHCP: `.11–.254` |
| LAN DHCP | `192.168.1.0/24` | R3 Fa5/0: `192.168.1.1`; servidor: `192.168.1.10` |
| R1–R2 | `192.168.0.0/30` | R1 G0/1: `192.168.0.1`; R2 G0/0: `192.168.0.2` |
| R1–R3 | `192.168.0.4/30` | R1 G0/2: `192.168.0.5`; R3 Fa0/0: `192.168.0.6` |
| R2–R4 | `192.168.0.8/30` | R2 G0/1: `192.168.0.9`; R4 G0/0: `192.168.0.10` |
| R3–R4 | `192.168.0.12/30` | R3 Fa1/0: `192.168.0.13`; R4 G0/1: `192.168.0.14` |
| AS2–R6 | `201.0.2.0/30` | R6 Fa1/0: `201.0.2.1`; R2 G0/2: `201.0.2.2` (IP de PAT) |

### AS3

| Segmento | Red | Interfaces y direcciones del maestro |
|---|---|---|
| VLAN 10 | `10.0.0.0/24` | R1 G0/0: `10.0.0.1`; PCs DHCP: `.11–.254` |
| Red de usuarios 2, conectada a R3 | `172.16.0.0/24` | R3 Fa4/0: `172.16.0.1`; PCs DHCP: `.11–.254` |
| LAN DHCP | `192.168.1.0/24` | R3 Fa5/0: `192.168.1.1`; servidor: `192.168.1.10` |
| R1–R2 | `192.168.0.0/30` | R1 G0/1: `.1`; R2 G0/0: `.2` |
| R1–R3 | `192.168.0.4/30` | R1 G0/2: `.5`; R3 Fa0/0: `.6` |
| R2–R4 | `192.168.0.8/30` | R2 G0/1: `.9`; R4 G0/0: `.10` |
| R3–R4 | `192.168.0.12/30` | R3 Fa1/0: `.13`; R4 G0/1: `.14` |
| AS3–R6 | `201.0.3.0/30` | R6 Fa2/0: `201.0.3.1`; R4 G0/2: `201.0.3.2` (IP de PAT) |

### AS4

| Segmento | Red | Interfaces y direcciones del maestro |
|---|---|---|
| VLAN 10 | `10.0.0.0/24` | R1 G0/0: `10.0.0.1`; PCs DHCP: `.11–.254` |
| Red de usuarios 2, conectada a R3 | `172.16.0.0/24` | R3 Fa4/0: `172.16.0.1`; PCs DHCP: `.11–.254` |
| LAN DHCP | `192.168.1.0/24` | R3 Fa5/0: `192.168.1.1`; servidor: `192.168.1.10` |
| R1–R2 | `192.168.0.0/30` | R1 G0/1: `.1`; R2 G0/0: `.2` |
| R1–R3 | `192.168.0.4/30` | R1 G0/2: `.5`; R3 Fa0/0: `.6` |
| R2–R4 | `192.168.0.8/30` | R2 G0/1: `.9`; R4 G0/0: `.10` |
| R3–R4 | `192.168.0.12/30` | R3 Fa1/0: `.13`; R4 G0/1: `.14` |
| AS4–R6 | `201.0.4.0/30` | R6 Fa3/0: `201.0.4.1`; R2 G0/2: `201.0.4.2` (IP de PAT) |

### AS99

| Segmento | Red | Interfaces y direcciones del maestro |
|---|---|---|
| R5–R6 | `200.99.0.0/30` | R5 G0/0: `200.99.0.1`; R6 Fa4/0: `200.99.0.2` |
| R6–R7 | `200.99.0.4/30` | R6 Fa5/0: `200.99.0.5`; R7 G0/0: `200.99.0.6` |
| R5–R7 | `200.99.0.8/30` | R5 G0/1: `200.99.0.9`; R7 G0/1: `200.99.0.10` |
| LAN DNS | `200.99.1.0/24` | R5: `200.99.1.1`; DNS: `200.99.1.10` |
| LAN WEB | `200.99.2.0/24` | R7: `200.99.2.1`; WEB: `200.99.2.10` |
| Enlace AS1–R6 | `201.0.1.0/30` | R6 Fa0/0: `.1`; AS1-R4 G0/2: `.2` |
| Enlace AS2–R6 | `201.0.2.0/30` | R6 Fa1/0: `.1`; AS2-R2 G0/2: `.2` |
| Enlace AS3–R6 | `201.0.3.0/30` | R6 Fa2/0: `.1`; AS3-R4 G0/2: `.2` |
| Enlace AS4–R6 | `201.0.4.0/30` | R6 Fa3/0: `.1`; AS4-R2 G0/2: `.2` |

En los cuatro AS cliente, R3 conecta la segunda red de usuarios en Fa4/0 (`172.16.0.1`), el enlace a R1 en Fa0/0 (`192.168.0.6`), el enlace a R4 en Fa1/0 (`192.168.0.13`) y la LAN DHCP en Fa5/0 (`192.168.1.1`). Cada interfaz pertenece a un segmento diferente. La red `172.16.0.0/24` es una LAN enrutada directamente por R3, separada de la VLAN 10 del switch.

#### Segmentación de la VLAN 10

![AS2: salida show vlan brief de S1](../evidencias/AS2/S1_show_vlan.png)

La salida muestra la VLAN 10 activa y los puertos de acceso asignados a esa red.

![AS3: salida show vlan brief de S1](../evidencias/AS3/S1_show_vlan.png)

![AS4: salida show vlan brief de S1](../evidencias/AS4/S1_show_vlan.png)


## Conectividad interna por AS

Cada AS cliente enruta internamente entre la VLAN 10, la LAN de usuarios conectada a R3 y la LAN del servidor DHCP. Los cuatro emplean las mismas redes internas. AS1 y AS3 ejecutan OSPF; AS2 y AS4 ejecutan RIP versión 2 con `no auto-summary`. Los servidores DHCP están en `192.168.1.10`.

**Diferentes IGP por AS.** El diseño usa OSPF en AS1, AS3 y AS99, y RIP versión 2 en AS2 y AS4. La coexistencia de protocolos satisface el requisito de emplear más de un IGP en la topología; dentro de cada AS, el protocolo asignado propaga las rutas internas y la ruta por defecto del borde. Entre AS, el intercambio se realiza con eBGP.

**DHCP relay.** Cada AS cliente tiene un servidor DHCP en `192.168.1.10`, separado de las redes de usuarios. Como las solicitudes iniciales DHCP son broadcasts y los routers no las reenvían entre subredes, R1 G0/0 y R3 Fa4/0 usan `ip helper-address 192.168.1.10` para reenviarlas como unicast. El servidor selecciona el pool según la red de origen indicada por el relay.

Las siguientes capturas muestran tablas de rutas, conectividad interna y asignaciones DHCP en cada AS. Cada imagen documenta únicamente los equipos y destinos visibles en ella.

### AS1

![AS1: tabla de rutas de R4](../evidencias/AS1/R4_show_ip_route.png)

En R4 se observan rutas OSPF internas y la ruta aprendida hacia el resumen de AS99.

![AS1: ping interno de PC0 a PC2](../evidencias/AS1/PC0_ping_PC2.png)

El ping responde desde un PC de la VLAN 10 hacia un PC de la LAN de usuarios conectada a R3 (`172.16.0.0/24`).

![AS1: concesión DHCP de PC0 en VLAN 10](../evidencias/AS1/PC0_ipconfig.png)

PC0 recibe una dirección de `10.0.0.0/24`, con gateway `10.0.0.1` y DNS `200.99.1.10`.

![AS1: concesión DHCP de PC2 en la red 172.16.0.0/24](../evidencias/AS1/PC2_ipconfig.png)

PC2 obtiene por DHCP una dirección de la LAN conectada a R3; la captura muestra gateway `172.16.0.1` y DNS `200.99.1.10`.

### AS2

![AS2: tabla de rutas de R1](../evidencias/AS2/R1_show_ip_route.png)

En R1 se distinguen las rutas RIP hacia las redes internas de AS2.

![AS2: pings internos desde PC0](../evidencias/AS2/PC0_ping_tests.png)

Los pings visibles alcanzan otro host de la VLAN 10, la LAN de usuarios conectada a R3 y el servidor DHCP.

![AS2: concesión DHCP de PC0 en VLAN 10](../evidencias/AS2/PC0_ipconfig.png)

PC0 recibe parámetros de DHCP para la red `10.0.0.0/24`.

![AS2: concesión DHCP de PC2 en la red 172.16.0.0/24](../evidencias/AS2/PC2_ipconfig.png)

La captura de PC2 confirma la concesión DHCP en `172.16.0.0/24`.

### AS3

![AS3: rutas y vecinos OSPF de R1](../evidencias/AS3/R1_ospf_neighbor_y_route.png)

R1 muestra las rutas internas y sus adyacencias OSPF.

![AS3: ping interno, acceso al DHCP y traceroute a PC2](../evidencias/AS3/PC0_ping_PC2_DHCP_tracert.png)

Los pings alcanzan PC2 en `172.16.0.0/24` y el servidor DHCP; el traceroute presenta la ruta hacia PC2.

![AS3: concesión DHCP de PC0 en VLAN 10](../evidencias/AS3/PC0_ipconfig.png)

La salida de PC0 confirma una dirección asignada en `10.0.0.0/24`.

![AS3: concesión DHCP de PC2 en la red 172.16.0.0/24](../evidencias/AS3/PC2_ipconfig.png)

La salida de PC2 confirma una dirección asignada en la LAN de usuarios conectada a R3.

### AS4

![AS4: tabla de rutas de R1](../evidencias/AS4/R1_show_ip_route.png)

La tabla de R1 presenta las rutas RIP internas de AS4.

![AS4: ping interno de PC0 a PC2](../evidencias/AS4/PC0_ping_PC2.png)

El ping responde desde un PC de la VLAN 10 hacia un PC de la LAN de usuarios conectada a R3 (`172.16.0.0/24`).

![AS4: concesión DHCP de PC0 en VLAN 10](../evidencias/AS4/PC0_ipconfig.png)

PC0 recibe una dirección del pool de `10.0.0.0/24`.

![AS4: concesión DHCP de PC2 en la red 172.16.0.0/24](../evidencias/AS4/PC2_ipconfig.png)

PC2 recibe una dirección del pool de la LAN conectada a R3.

## Interconexión entre AS

Cada borde de cliente se conecta a R6 mediante un enlace /30 y una sesión eBGP: AS1-R4, AS2-R2, AS3-R4 y AS4-R2. R6 anuncia el resumen `200.99.0.0/16`, que cubre las redes de AS99. Cada borde usa una ruta estática por defecto hacia R6 y la distribuye dentro de su AS mediante su IGP. Las redes privadas de los clientes no se anuncian fuera de sus AS; el tráfico saliente se traduce a la IP del enlace público correspondiente.

![Resumen eBGP de R6](../evidencias/extremo-a-extremo/R6_bgp_summary.png)

La tabla de vecinos de R6 muestra las cuatro sesiones eBGP establecidas con los routers de borde de AS1–AS4.

![Tabla de rutas de R6](../evidencias/extremo-a-extremo/R6_show_ip_route.png)

La tabla muestra el agregado estático `200.99.0.0/16` hacia Null0, las rutas OSPF específicas hacia DNS y WEB, y los cuatro enlaces a clientes como rutas conectadas. No aparecen los prefijos internos de los clientes.

#### Rutas y traducciones NAT en los bordes

![Rutas y traducciones NAT en AS1-R4](../evidencias/extremo-a-extremo/AS1_R4_route_nat.png)

R4 aprende por BGP `200.99.0.0/16` y muestra traducciones PAT hacia DNS y WEB con origen privado `10.0.0.12`.

![Rutas y traducciones NAT en AS2-R2](../evidencias/extremo-a-extremo/AS2_R2_route_nat.png)

R2 muestra rutas RIP internas, la ruta por defecto, el prefijo BGP de AS99 y traducciones PAT de `10.0.0.12` hacia DNS y WEB.

![Rutas y traducciones NAT en AS3-R4](../evidencias/extremo-a-extremo/AS3_R4_route_nat.png)

R4 muestra la ruta por defecto, aprende el agregado de AS99 por BGP y traduce tráfico de `10.0.0.12` hacia DNS y WEB.

![Rutas y traducciones NAT en AS4-R2](../evidencias/extremo-a-extremo/AS4_R2_route_nat.png)

R2 muestra rutas RIP, la ruta por defecto, el prefijo BGP de AS99 y traducciones PAT de `10.0.0.12` hacia DNS y WEB.

![Conectividad entre los bordes de AS1 y AS2](../evidencias/extremo-a-extremo/AS1_PC0_entre_AS.png)

Desde AS1, el ping a la IP pública del borde AS2 (`201.0.2.2`) recibe cuatro respuestas. El traceroute a `10.0.0.11` muestra ese destino como primer y único salto, consistente con la resolución local de un prefijo solapado.

**NAT por solapamiento de direcciones IP.** Los cuatro AS de clientes reutilizan las redes `10.0.0.0/24`, `172.16.0.0/24` y segmentos dentro de `192.168.0.0/16`. Si esos prefijos se propagaran al proveedor, R6 no podría distinguir qué AS contiene cada red repetida. Las ACL de NAT permiten esos dos segmentos de usuarios y el espacio `192.168.0.0/16`; PAT traduce las fuentes permitidas a la IP pública del borde (`201.0.k.2`). Las capturas NAT muestran traducciones activas desde `10.0.0.12` hacia DNS y WEB en los cuatro AS.

**eBGP e iBGP en Packet Tracer.** Los AS de clientes y el AS99 son sistemas autónomos distintos, por eso los bordes de cliente establecen eBGP con R6. R6 anuncia `200.99.0.0/16` y los bordes no anuncian las redes internas. Debido a las limitaciones de soporte de iBGP en Packet Tracer, R6 es el único router BGP de AS99; R5 y R7 aprenden por OSPF las redes de servicios y los enlaces públicos necesarios para responder a los clientes.

**Alcance entre clientes.** Los hosts internos de distintos clientes no tienen conectividad directa. Como comparten direccionamiento, un destino de la red local, como `10.0.0.11`, se resuelve en el propio AS de origen. Además, los bordes no anuncian redes internas por BGP y PAT no publica automáticamente una conexión entrante hacia un host interno. La prueba ilustrada muestra que sí es alcanzable la IP pública del borde AS2 (`201.0.2.2`) desde AS1. Publicar un servicio interno requeriría una traducción estática o un reenvío de puertos.

## Acceso a servicios

El DNS del AS99 (`200.99.1.10`) contiene el registro A `www.iap.com → 200.99.2.10`. El servidor WEB está en `200.99.2.10` y su servicio HTTP está activo. Las capturas siguientes muestran, desde un PC de la VLAN 10 en cada AS, los parámetros recibidos por DHCP, la resolución del nombre y el acceso al sitio WEB.

### Acceso a DNS y WEB desde PC0 de cada AS

#### AS1

![AS1: DHCP, nslookup y ping al servidor WEB](../evidencias/extremo-a-extremo/AS1_PC0_dns_ping.png)

La consulta resuelve `www.iap.com` a `200.99.2.10`; el ping por nombre termina con tres respuestas de cuatro tras un timeout inicial.

![AS1: página HTTP abierta por dominio](../evidencias/extremo-a-extremo/AS1_PC0_web.png)

El navegador carga la página predeterminada desde `http://www.iap.com`.

#### AS2

![AS2: DHCP, nslookup y ping al servidor WEB](../evidencias/extremo-a-extremo/AS2_PC0_dns_ping.png)

La salida de PC0 resuelve `www.iap.com` a `200.99.2.10` y muestra cuatro respuestas al ping.

![AS2: página HTTP abierta por dominio](../evidencias/extremo-a-extremo/AS2_PC0_web.png)

PC0 de AS2 carga la página predeterminada de Packet Tracer desde la URL HTTP indicada.

#### AS3

![AS3: DHCP, nslookup y ping al servidor WEB](../evidencias/extremo-a-extremo/AS3_PC0_dns_ping.png)

La consulta DNS de PC0 devuelve `200.99.2.10`; el ping al destino resuelto recibe cuatro respuestas.

![AS3: página HTTP abierta por dominio](../evidencias/extremo-a-extremo/AS3_PC0_web.png)

La captura del navegador muestra la página predeterminada servida para `http://www.iap.com` en AS3.

#### AS4

![AS4: DHCP, nslookup y ping al servidor WEB](../evidencias/extremo-a-extremo/AS4_PC0_dns_ping.png)

PC0 resuelve el dominio al servidor `200.99.2.10` y completa cuatro respuestas al ping por nombre.

![AS4: página HTTP abierta por dominio](../evidencias/extremo-a-extremo/AS4_PC0_web.png)

El navegador de AS4 presenta la página predeterminada solicitada por `http://www.iap.com`.

## Pruebas

Las capturas de las secciones anteriores documentan conectividad interna y acceso a servicios. Las pruebas de interconexión se presentan a continuación.

| Prueba | Resultado que muestran las capturas |
|---|---|
| Enrutamiento interno | Las tablas de rutas muestran OSPF en AS1/AS3 y RIP en AS2/AS4; los pings visibles confirman conectividad entre los equipos indicados en cada captura. |
| DHCP | Las capturas de configuración IP muestran direcciones de las redes de usuarios, gateway y DNS asignados. |
| eBGP | R6 muestra cuatro vecinos establecidos con AS1–AS4. |
| Rutas entre AS | La tabla de R6 muestra el resumen de AS99 y las redes de enlace de clientes como conectadas, sin prefijos internos de clientes. |
| NAT/PAT | Las tablas de los cuatro bordes muestran traducciones de origen de clientes hacia DNS/WEB usando la dirección del enlace público correspondiente. |
| DNS y WEB | Desde un PC0 de cada AS, las capturas muestran la resolución de `www.iap.com`, respuestas al ping al servidor WEB y la carga de la página por HTTP. |
| Conectividad entre clientes | Desde AS1, el ping a la dirección pública del borde AS2 responde; el traceroute al destino interno coincidente termina en la red local de origen. |
