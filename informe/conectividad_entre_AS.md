# Conectividad entre AS de clientes

Responsable: Emmanuel Castaño Sepúlveda. Archivo: `packet-tracer/Entrega2_IAP.pkt`.

## Pregunta

¿Puede un host de un AS de cliente comunicarse con un host de otro AS de cliente? Por ejemplo, PC0 del AS1 (10.0.0.11) con PC0 del AS2 (también 10.0.0.11).

## Respuesta corta

No, para los hosts internos. Sí, para la IP pública del router de borde de cada AS (201.0.k.2).

## Por qué

1. **Las direcciones internas son las mismas en los cuatro AS.** El enunciado fija 10.0.0.0/24 y 172.16.0.0/24 en todos los clientes. Si PC0 del AS1 hace `ping 10.0.0.11`, el destino pertenece a su propia red 10.0.0.0/24: el paquete nunca sale del AS1 y lo responde el host local que tiene esa IP.
2. **Las redes internas no se anuncian fuera del AS.** Cada borde usa eBGP solo para aprender 200.99.0.0/16 de R6; no anuncia 10.0.0.0/24, 172.16.0.0/24 ni 192.168.0.0/16. R6 no tiene ninguna ruta hacia esas redes, así que un paquete dirigido a ellas desde otro AS se descarta en R6.
3. **El NAT overload solo traduce conexiones iniciadas desde adentro.** Cuando un host sale, el borde crea una entrada en la tabla NAT (10.0.0.11:puerto ↔ 201.0.k.2:puerto). Un paquete que llega desde afuera sin una entrada previa no tiene a quién traducirse, y el borde lo responde él mismo o lo descarta.
4. **Lo que sí es alcanzable es la IP pública de otro borde.** Ejemplo: PC0 del AS1 hace `ping 201.0.2.2`.
   - Ida: AS1-R4 no conoce 201.0.2.0/30, así que usa su ruta por defecto hacia R6 (201.0.1.1) y traduce el origen a 201.0.1.2. R6 tiene 201.0.2.0/30 conectada y entrega el paquete a AS2-R2.
   - Vuelta: AS2-R2 responde a 201.0.1.2 por su ruta por defecto hacia R6. R6 tiene 201.0.1.0/30 conectada y lo entrega a AS1-R4, que deshace la traducción y lo envía a PC0.

## Pruebas en el archivo maestro

Antes de probar, presiona **Fast Forward Time** varias veces para que converjan OSPF, RIP y BGP, y renueva la IP de los PCs (Desktop → IP Configuration → Static y de nuevo DHCP).

| # | Dónde | Comando | Resultado esperado | Captura |
|---|---|---|---|---|
| 1 | R6 | `show ip bgp summary` | 4 vecinos (201.0.1.2 a 201.0.4.2) establecidos | `R6_bgp_summary.png` |
| 2 | R6 | `show ip route` | Solo redes 200.99.x.x y 201.0.k.0/30. No aparecen 10.0.0.0, 172.16.0.0 ni 192.168.x.x | `R6_show_ip_route.png` |
| 3 | AS1-PC0 | `ping 201.0.2.2` | Responde: la IP pública del borde del AS2 es alcanzable | `AS1_PC0_entre_AS.png` |
| 4 | AS1-PC0 | `tracert 10.0.0.11` | Un solo salto o respuesta directa: se queda dentro del AS1 (lo responde el PC del propio AS1) | `AS1_PC0_entre_AS.png` |
| 5 | AS2-R2 | `show ip nat translations` (justo después de la prueba 3) | Ninguna traducción con 201.0.1.2 como destino externo: el borde respondió el ping él mismo y no lo reenvió a ningún host interno | `AS2_R2_route_nat.png` |

Las capturas van en `evidencias/extremo-a-extremo/`.

## Texto para el informe

Los cuatro AS de clientes comparten el mismo direccionamiento interno (10.0.0.0/24 y 172.16.0.0/24), fijado por el enunciado. Por eso no es posible que un host de un AS alcance un host interno de otro AS: un destino 10.0.0.x siempre se resuelve dentro del propio AS, las redes internas nunca se anuncian por BGP (R6 no tiene rutas hacia ellas) y el NAT overload de cada borde solo permite conexiones iniciadas desde adentro. Lo único alcanzable entre clientes es la IP pública de cada borde (201.0.k.2), porque R6 tiene conectadas todas las redes 201.0.k.0/30 y cada borde envía por su ruta por defecto lo que no conoce. Si en algún caso se necesitara ofrecer un servicio de un cliente hacia otro AS, la solución sería un NAT estático o un reenvío de puerto en el borde, que publica un host interno detrás de la IP pública.
