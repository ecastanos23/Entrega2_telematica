# Entrega 2 – Interconexión de sistemas autónomos y acceso a servicios

Internet, Arquitectura y Protocolos (Telemática) 2026-2 — EAFIT.

Implementación en Packet Tracer de 4 sistemas autónomos de clientes (AS1–AS4) conectados a un AS99 proveedor de servicios (DNS y WEB). Los clientes obtienen IP por DHCP, resuelven nombres con el DNS del AS99 y acceden al servidor WEB por dominio.

## Integrantes y responsabilidades

| Integrante | AS | IGP | Eje común |
|---|---|---|---|
| Emmanuel Castaño Sepúlveda | AS1 | OSPF | Interconexión entre AS (NAT y eBGP) e integración del `.pkt` |
| Viviana Arango Tabares | AS2 | RIP | Informe y entrega |
| Jerónimo Contreras Sierra | AS3 | OSPF | AS99 (R5, R6, R7) |
| Simón Mazo Gómez | AS4 | RIP | Servicios DNS/WEB y pruebas |

## Estructura del repositorio

```
Entrega2_telematica/
├── packet-tracer/
│   ├── Entrega2_IAP.pkt      # Archivo maestro integrado (solo lo modifica el integrador)
│   └── borradores/           # Archivos de trabajo de cada integrante (ASx_nombre.pkt)
├── configs/
│   ├── AS1.txt ... AS4.txt   # show running-config de cada router del AS
│   ├── AS99.txt              # R5, R6 y R7
│   └── plantilla-borde.txt   # NAT + eBGP + ruta por defecto para los routers de borde
├── evidencias/
│   ├── AS1/ ... AS4/         # Capturas de pruebas internas
│   ├── AS99/
│   └── extremo-a-extremo/    # ping, nslookup y HTTP desde cada AS
└── informe/                  # Informe final (fuente y PDF)
```

## Flujo de trabajo

1. Cada integrante trabaja su AS en `packet-tracer/borradores/` siguiendo el plan de direccionamiento acordado.
2. Al terminar, guarda el `show running-config` de cada router en `configs/ASx.txt` (un bloque por router, con el nombre del router como encabezado).
3. Las capturas van en `evidencias/ASx/` con nombres descriptivos (por ejemplo `R1_show_ip_route.png`, `PC1_ipconfig.png`).
4. El integrador aplica los scripts de `configs/` sobre `packet-tracer/Entrega2_IAP.pkt`. Packet Tracer no fusiona archivos, por eso la integración se hace con los comandos y no copiando archivos `.pkt`.
5. Todos usan la misma versión de Packet Tracer: `[...]`.

## Plan de direccionamiento (resumen)

| Segmento | Red | Notas |
|---|---|---|
| VLAN 10 (cada AS de cliente) | 10.0.0.0/24 | Fija por enunciado, gateway en R1 |
| VLAN 20 (cada AS de cliente) | 172.16.0.0/24 | Fija por enunciado, gateway en R3 |
| LAN del servidor DHCP | 192.168.1.0/24 | Junto a R3 |
| Enlaces internos de cada AS | 192.168.0.0/30 – 192.168.0.12/30 | R1–R2, R1–R3, R2–R4, R3–R4 |
| Enlaces internos del AS99 | 200.99.0.0/30 – 200.99.0.8/30 | R5–R6, R6–R7, R5–R7 |
| LAN DNS (AS99) | 200.99.1.0/24 | Servidor DNS 200.99.1.10 |
| LAN WEB (AS99) | 200.99.2.0/24 | Servidor WEB 200.99.2.10 (`www.iap.com`) |
| Enlace ASk – R6 | 201.0.k.0/30 | R6 .1, borde del AS .2 (IP usada por el NAT) |

## Decisiones de diseño

- **Direcciones solapadas:** los 4 clientes usan las mismas redes internas, así que cada router de borde hace NAT overload hacia su IP del enlace con R6. Por eso los AS de clientes no pueden alcanzar las redes internas de otros AS.
- **Interconexión:** eBGP entre el borde de cada AS de cliente y R6. Por las limitaciones de iBGP en Packet Tracer, R6 es el único router BGP del AS99; R5 y R7 aprenden las rutas por OSPF.
- **IGP:** OSPF en AS1, AS3 y AS99; RIP en AS2 y AS4 (el enunciado prohíbe un único IGP en toda la topología).
- **DHCP:** un servidor por AS de cliente, con `ip helper-address` en las interfaces gateway de R1 y R3.
