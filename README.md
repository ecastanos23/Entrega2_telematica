# Entrega 2 – Interconexión de sistemas autónomos y acceso a servicios

Internet, Arquitectura y Protocolos (Telemática) 2026-2 — EAFIT.

Implementación en Packet Tracer de 4 sistemas autónomos de clientes (AS1–AS4) conectados a un AS99 proveedor de servicios (DNS y WEB). Los clientes obtienen IP por DHCP, resuelven nombres con el DNS del AS99 y acceden al servidor WEB por dominio.

**Archivo de entrega:** `packet-tracer/Entrega2_IAP.pkt` (Packet Tracer 9.0.1).

## Integrantes y responsabilidades

| Integrante | AS | IGP | Eje común |
|---|---|---|---|
| Emmanuel Castaño Sepúlveda | AS1 | OSPF | Interconexión entre AS (NAT y eBGP) e integración del `.pkt` |
| Viviana Arango Tabares | AS2 | RIP | Informe y entrega |
| Jerónimo Contreras Sierra | AS3 | OSPF | AS99 (R5, R6, R7) |
| Simón Mazo Gómez | AS4 | RIP | Servicios DNS/WEB y pruebas |

## Estado

| Componente | Estado |
|---|---|
| AS1, AS2, AS3, AS4 | Configurados e integrados en el maestro |
| AS99 (R5, R6, R7, DNS, WEB) | Integrado; R6 con los 4 vecinos eBGP |
| Registro DNS `www.iap.com` | Cargado en el servidor DNS del maestro (registro A → 200.99.2.10) |
| Pruebas de extremo a extremo | Completas en `evidencias/extremo-a-extremo/` (los 4 AS) |
| Informe | En `informe/` |

## Estructura del repositorio

```
Entrega2_telematica/
├── packet-tracer/
│   ├── Entrega2_IAP.pkt      # Archivo maestro integrado (solo lo modifica el integrador)
│   └── borradores/           # Archivos de trabajo de cada integrante
├── configs/
│   ├── AS1.md ... AS4.md     # show running-config de cada AS
│   ├── AS99.md               # R5, R6, R7 y servidores
│   └── plantilla-borde.md    # NAT + eBGP + ruta por defecto para los routers de borde
├── evidencias/
│   ├── AS1/ ... AS4/         # Pruebas internas de cada AS
│   ├── AS99/
│   ├── prueba-bgp-nat/       # Prueba previa de eBGP y NAT en Packet Tracer 9.0
│   └── extremo-a-extremo/    # ping, nslookup, HTTP y conectividad entre AS desde el maestro
└── informe/                  # Informe final y análisis de conectividad entre AS
```

## Topología del archivo maestro

Cada AS de cliente tiene los mismos equipos y el mismo cableado. En el lienzo, los equipos se llaman `ASk-R1`, `ASk-S1`, `ASk-PC0`, etc.

| Equipo | Modelo | Puertos |
|---|---|---|
| R1 | Cisco 2911 | G0/0 a S1 G0/1 (VLAN 10), G0/1 a R2, G0/2 a R3 |
| R2 | Cisco 2911 | G0/0 a R1, G0/1 a R4, G0/2 a R6 (solo en AS2 y AS4, que tienen el borde en R2) |
| R3 | PT-Router con PT-ROUTER-NM-1CFE | Fa0/0 a R1, Fa1/0 a R4, Fa4/0 a PC2 (172.16.0.0/24), Fa5/0 al servidor DHCP |
| R4 | Cisco 2911 | G0/0 a R2, G0/1 a R3, G0/2 a R6 (solo en AS1 y AS3, que tienen el borde en R4) |
| S1 | Cisco 2960 | G0/1 a R1, Fa0/1 a PC0, Fa0/2 a PC1, todos en VLAN 10 |

| Puerto de R6 | Conecta con | Red |
|---|---|---|
| Fa0/0 | AS1-R4 G0/2 | 201.0.1.0/30 |
| Fa1/0 | AS2-R2 G0/2 | 201.0.2.0/30 |
| Fa2/0 | AS3-R4 G0/2 | 201.0.3.0/30 |
| Fa3/0 | AS4-R2 G0/2 | 201.0.4.0/30 |
| Fa4/0 | R5 G0/0 | 200.99.0.0/30 |
| Fa5/0 | R7 G0/0 | 200.99.0.4/30 |

R5 y R7 se unen por G0/1 (200.99.0.8/30). El DNS está en R5 G0/2 y el WEB en R7 G0/2.

## Plan de direccionamiento (resumen)

| Segmento | Red | Notas |
|---|---|---|
| VLAN 10 (cada AS de cliente) | 10.0.0.0/24 | Fija por enunciado, gateway en R1 (10.0.0.1) |
| Red de PC2 (cada AS de cliente) | 172.16.0.0/24 | Fija por enunciado, gateway en R3 (172.16.0.1) |
| LAN del servidor DHCP | 192.168.1.0/24 | Servidor 192.168.1.10, gateway R3 (192.168.1.1) |
| Enlaces internos de cada AS | 192.168.0.0/30 – 192.168.0.12/30 | R1–R2, R1–R3, R2–R4, R3–R4 |
| Enlaces internos del AS99 | 200.99.0.0/30 – 200.99.0.8/30 | R5–R6, R6–R7, R5–R7 |
| LAN DNS (AS99) | 200.99.1.0/24 | Servidor DNS 200.99.1.10 |
| LAN WEB (AS99) | 200.99.2.0/24 | Servidor WEB 200.99.2.10 (`www.iap.com`) |
| Enlace ASk – R6 | 201.0.k.0/30 | R6 .1, borde del AS .2 (IP pública del NAT) |

## Decisiones de diseño

- **Direcciones solapadas:** los 4 clientes usan las mismas redes internas, así que cada router de borde hace NAT overload (PAT) hacia su IP del enlace con R6. El AS99 nunca ve las redes internas y las respuestas siempre vuelven al AS correcto.
- **Interconexión:** eBGP entre el borde de cada AS de cliente y R6. R6 anuncia un único resumen 200.99.0.0/16 (con ruta a Null0). Por las limitaciones de iBGP en Packet Tracer, R6 es el único router BGP del AS99; R5 y R7 aprenden por OSPF los enlaces 201.0.k.0/30.
- **Ruta por defecto:** cada borde tiene `ip route 0.0.0.0 0.0.0.0 201.0.k.1` y la anuncia en su IGP con `default-information originate`, en lugar de redistribuir BGP dentro del AS.
- **IGP:** OSPF en AS1, AS3 y AS99; RIP versión 2 con `no auto-summary` en AS2 y AS4 (el enunciado prohíbe un único IGP en toda la topología).
- **DHCP:** un servidor por AS de cliente, con `ip helper-address 192.168.1.10` en el gateway de cada red de clientes (R1 G0/0 y R3 Fa4/0).
- **Conectividad entre clientes:** un host no puede alcanzar los hosts internos de otro AS; solo la IP pública de su borde. El análisis completo está en `informe/conectividad_entre_AS.md`.

## Integración

Las configuraciones se tomaron de los borradores de cada integrante y se cargaron en el maestro sin cambiar su contenido, salvo estos ajustes:

| Ajuste | Motivo |
|---|---|
| Hostname de los equipos de AS2, AS3 y AS4 igual a su nombre en el lienzo (`AS2-R1`, `AS3-R4`, etc.); AS1 y AS99 conservan `R1`–`R4` y `R5`–`R7` | Identificar el AS en las capturas |
| R3 de AS2 y AS4: Fa0/0↔Fa4/0 y Fa1/0↔Fa5/0 | En los borradores de AS2 y AS4, R3 estaba cableado distinto; en el maestro todos los R3 usan Fa0/0 a R1, Fa1/0 a R4, Fa4/0 a PC2 y Fa5/0 al DHCP |
| S1 del AS2: G0/1 agregado a la VLAN 10 | En el maestro, R1 se conecta a S1 por G0/1 |
| AS2-R2: se quitaron las líneas `network` de `router bgp 2` (10.0.0.0/24, 172.16.0.0/24, 192.168.0.x/30 y 192.168.1.0) | El borrador anunciaba las redes internas del AS2 por BGP. R6 las reenviaba a los otros bordes y, como eBGP (distancia 20) le gana a RIP (120) y a OSPF (110), los demás AS enviaban hacia el AS2 el tráfico de sus propias redes internas. Con NAT, los bordes no deben anunciar redes internas |
| R6: `neighbor 201.0.k.2 remote-as k` para k = 1 a 4 | Sesiones eBGP con los 4 bordes |
| Servidor DNS: servicio On y registro A `www.iap.com → 200.99.2.10` | Resolución del dominio del WEB (no estaba en ningún borrador) |
| Servidores DHCP de AS2, AS3 y AS4 con IP fija y los pools de su borrador | Igual que en cada borrador |

El servicio HTTP del servidor `WEB` está activo con la página por defecto de Packet Tracer.

**Verificado en el maestro:** R6 con los 4 vecinos eBGP `Up` y adyacencias OSPF `FULL` con R5 y R7; AS1-PC0 (VLAN 10) y AS4-PC2 (red 172.16.0.0/24) obtienen IP por DHCP, resuelven `www.iap.com` y hacen ping a 200.99.2.10.

## Cómo verificar el archivo maestro

1. Abre `packet-tracer/Entrega2_IAP.pkt` y presiona **Fast Forward Time** varias veces para que converjan OSPF, RIP y BGP.
2. En cada PC: *Desktop → IP Configuration* → **DHCP** (si ya estaba en DHCP, cambia a Static y vuelve a DHCP).
3. Ejecuta las pruebas y guarda las capturas en `evidencias/extremo-a-extremo/`:

| Prueba | Dónde | Comando | Resultado esperado |
|---|---|---|---|
| eBGP | R6 | `show ip bgp summary` | 4 vecinos establecidos |
| Rutas | Borde de cada AS | `show ip route` | `B 200.99.0.0/16` y `S* 0.0.0.0/0` |
| Conectividad | Un PC de cada AS | `ping 200.99.1.10` y `ping 200.99.2.10` | Responden |
| DNS | Un PC de cada AS | `nslookup www.iap.com` | Devuelve 200.99.2.10 |
| HTTP | Un PC de cada AS | *Desktop → Web Browser* → `http://www.iap.com` | Carga la página del WEB |
| NAT | Borde de cada AS | `show ip nat translations` | IP interna traducida a 201.0.k.2 |
| Entre AS | AS1-PC0 | `ping 201.0.2.2` y `tracert 10.0.0.11` | El primero responde; el segundo se queda dentro del AS1 |

## Evidencias de extremo a extremo

Tomadas en el archivo maestro, en `evidencias/extremo-a-extremo/`:

| Archivo | Qué muestra |
|---|---|
| `topologia_maestro.png` | Topología completa con los 5 AS |
| `R6_bgp_summary.png` | R6 con los 4 vecinos eBGP establecidos |
| `R6_show_ip_route.png` | Tabla de R6: solo redes 200.99.x.x y 201.0.k.0/30, ninguna red interna de los clientes |
| `ASk_PC0_dns_ping.png` (k = 1 a 4) | IP por DHCP, `nslookup www.iap.com` → 200.99.2.10 y ping por nombre al WEB |
| `ASk_PC0_web.png` (k = 1 a 4) | Navegador del PC cargando `http://www.iap.com` |
| `AS1_R4_route_nat.png`, `AS2_R2_route_nat.png`, `AS3_R4_route_nat.png`, `AS4_R2_route_nat.png` | Tabla de rutas del borde (`B 200.99.0.0/16`, `S* 0.0.0.0/0`) y traducciones NAT de DNS (UDP 53) y HTTP (TCP 80) hacia 201.0.k.2 |
| `AS1_PC0_entre_AS.png` | `ping 201.0.2.2` responde (IP pública del borde del AS2) y `tracert 10.0.0.11` se queda dentro del AS1 |

## Flujo de trabajo

1. Cada integrante trabaja en `packet-tracer/borradores/` y documenta su AS en `configs/ASk.md`.
2. Las capturas van en `evidencias/ASk/` con nombres descriptivos (por ejemplo `R1_show_ip_route.png`, `PC0_ipconfig.png`).
3. Solo el integrador modifica `packet-tracer/Entrega2_IAP.pkt`. Packet Tracer no fusiona archivos y un `.pkt` no se puede combinar en Git.
4. Todos usan Packet Tracer **9.0.1**.
