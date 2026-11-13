# Tarea 3 - faznet

## Integrantes

| Nombre | Número de alumno |
|---|---|
| Franco Duarte | 25663674 |
| Vicente Del Fierro | 19639562 |

---

## Parte 1: Construcción de la red de Fazbear Entertainment

### Enlaces entre routers

Indiquen las direcciones que eligieron para los enlaces entre routers.

| Enlace | Red / máscara | IP en el primer router | IP en el segundo router |
|---|---|---|---|
| R-Oficina ↔ R-HQ | 10.0.0.0/30 (255.255.255.252) | 10.0.0.1 | 10.0.0.2 |
| R-HQ ↔ R-Pizzaplex | 10.0.0.4/30 (255.255.255.252) | 10.0.0.5 | 10.0.0.6 |

## Parte 2: Simulación en la red de Fazbear Entertainment

### 2.1

**Dispositivos finales, en orden:**

1. PC del Guardia: envía la consulta a DNS HQ.
2. Servidor DNS HQ: consulta al DNS del Pizzaplex.
3. Servidor DNS del Pizzaplex: responde a DNS HQ, que entrega
   la respuesta al PC del Guardia.

**IP obtenida por el PC del Guardia:**

172.16.21.3, correspondiente al Servidor Web.

**¿Por qué el Servidor DNS HQ no podía responder solo con sus registros?**

Porque el registro A del servidor DNS HQ no tiene comunicacion directa con el Servidor Web, sino que se conecta con un registro NS al Servidor DNS de Pizzaplex

### 2.2

**IP de origen del primer mensaje DNS que llega al DNS del Pizzaplex:** 10.19.83.2 (Server DNS HQ)

**Explicación:**

Esta consulta DNS llega desde el servidor DNS de HQ, porque este no tiene el registro A del sitio solicitado. Su registro NS indica que debe consultar al DNS de Pizzaplex, y el registro A de ns.pizzaplex.fazbear.com le proporciona la IP de ese servidor. Por eso la IP de origen es 10.19.83.2, correspondiente al DNS HQ.

### 2.3

**Predicción** _(escrita antes de simular)_**:** Esperamos que el PC del Guardia no pueda resolver la dirección del sitio. El registro NS indica que el DNS HQ debe consultar a ns.pizzaplex.fazbear.com, pero al no haber registro A, HQ no tiene la IP de ese servidor para enviarle la consulta.

**Lo que ocurrió realmente** _(hasta dónde llega la resolución y qué muestra el navegador)_**:**

La consulta llegó al DNS HQ, donde observamos únicamente una PDUde entrada, sin una consulta de salida hacia el DNS Pizzaplex. La resolución no continuó y el navegador mostró “Host Name Unresolved”.


**¿Fue correcta la predicción? Si no, ¿qué entendieron mal?**

Fue correcta, el servidor de HQ no fue capaz de establecer la conexion ya que no tenia la IP del servidor responsable, solo con su nombre.

### 2.4

**Protocolo para enviar:** SMTP, luego de resolver la direccion del servidor de correo, la consulta es SMTP

**Protocolo para recibir:** POP3. La consulta al recibir tiene protocolo POP3

**¿El DNS HQ consultó a otro servidor para resolver `mail.fazbear.com`? Comparación con la 2.1:**

No, ya que tiene el registro A de mail.fazbear.com, que indica la IP del servidor de correo. En 2.1, no se tenia el registro A de ns.pizzaplex.fazbear,com, por lo que habia que consultar primero la DNS para resolver la consulta.

---

## Parte 3: Troubleshooting en Fazbear's Fright

| N° | Síntoma observado | Cómo la encontraron | Dispositivo y configuración incorrecta | Corrección aplicada |
|---|---|---|---|---|
| 1 | DNS incorrecto (10.30.0.20) en PCs de Boleteria | Pingeando la web directamente con su direccion desde Boleteria y luego con `nslookup` sin exito | Configuración DHCP en R-Atraccion | Corregida la configuración de DHCP en R-Atraccion para entregar el servidor DNS correcto (10.30.0.2) |
| 2 | Dirección IP de fazbearsfright.com incorrecta en DNS (10.30.0.30) | Intentado hacer Ping a www.fazbearsfright.com, la consola muestra una IP que no corresponde a la del servidor  | Dirección incorrecta en Servidor DNS | Se asigna la dirección IP que corresponde (10.30.0.3) al servidor web en el servidor DNS |
| 3 | Los computadores de boletería pueden acceder al servidor de registros | Se generan pings exitosos en la IP del servidor de registros (10.30.0.4) desde ambos PCs de boletería | Firewall del server de registros | Se modificó la regla de Deny para la subred de 192.168.33.0 para que se excluya a los PCs de boletería  |
| 4 | La PC de Seguridad no puede acceder a los servidores | Se generan timeouts al hacer ping a los servidores de DNS, Web y de los Registros | Tabla de Direccionamiento CEF del Router de Servidores | Se añade la ruta de retorno desde (192.168.34.0) hacia (10.255.0.1) en la configuración del router |
| 5 | El Gateway de PC Técnico está mal asignado a la última dirección disponible (10.30.0.254) | Revisando los Gateways de todos los dispositivos en la red, El del PC Técnico no cuadraba con la del resto de dispositivos (10.30.0.1) | Gateway PC Técnico | Se asigna la primera Dirección de la subred al Gateway (10.30.0.1) |

---

## Parte 4: Simulación en Fazbear's Fright

### 4.1

**Dispositivo donde se descarta el paquete:**
[En el Servidor de Registro]
**Capa del modelo OSI:**
[Layer 3: Red IP]
**IP de origen observada:**
[192.168.33.3]
**¿Por qué el paquete coincide con la regla del firewall (dirección y wildcard)?:**
Porque forma parte de la subred 192.168.33.XXX, todas las IPs de dispositivos provenientes desde esa subred son bloqueadas por el wildcard (0.0.0.255)

### 4.2

**¿Llega el echo request al Servidor Web?:**
 No llega
**Dispositivo donde se pierde el paquete y por qué:**
Se pierde en el Router de Atracciones, No encuentra la dirección de destino en la tabla CEF
**Falla relacionada (N° de su tabla):**
 4, No se encontró la dirección por estar ausente también la dirección de retorno en el router de Servidores
 
### 4.3

**IP a la que se envía la consulta DNS:**
 Se envía a 10.30.0.20
**Dispositivo hasta donde llega:**
 Al no existir la dirección en la tabla de direcciones, solo llega al router de servidores antes de ser descartado
**¿Por qué nunca obtiene respuesta?:**
Porque el paquete nunca llegó a su destino
### 4.4

**Fallas que producen el mismo síntoma (N° de su tabla):**

**Cómo distinguirlas usando `ping`, `nslookup` y/o el modo simulación:**
