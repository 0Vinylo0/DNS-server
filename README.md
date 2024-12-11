# DNS Server con Bind9

# Protocolo DNS: Funcionamiento, Puertos, y Protocolos de Transporte

El **Sistema de Nombres de Dominio (DNS)** es un protocolo fundamental en Internet que traduce los **nombres de dominio** (como `www.ejemplo.com`) en **direcciones IP** que los dispositivos utilizan para comunicarse. Esto permite que los usuarios utilicen nombres fáciles de recordar, en lugar de direcciones numéricas.

## Funcionamiento del Protocolo DNS

El DNS se basa en una estructura jerárquica de servidores que proporcionan las respuestas a las consultas realizadas por los clientes (usualmente conocidos como **resolutores**). Los pasos básicos para una consulta DNS son los siguientes:

1. **Consulta Recursiva**: Un cliente (o resolutor) envía una consulta a su servidor DNS configurado para resolver un nombre de dominio. Si el servidor no tiene la respuesta, realiza consultas adicionales para encontrar la respuesta.

2. **Servidor Raíz**: Si el servidor DNS no tiene la respuesta en caché, consultará un **servidor raíz** para obtener la dirección de los servidores de nombres de nivel superior (**TLD**, como `.com`, `.org`, etc.).

3. **Servidor TLD**: Luego, el servidor DNS contacta a los servidores TLD que le proporcionan la dirección del servidor autoritativo que tiene la información del dominio específico.

4. **Servidor Autoritativo**: Finalmente, el servidor DNS contacta al **servidor autoritativo** del dominio que tiene la respuesta y devuelve la dirección IP solicitada al cliente.

## Puertos Utilizados por DNS

- **Puerto 53**: El DNS utiliza el **puerto 53** tanto para el **protocolo UDP** como para **TCP**.
  - **UDP/53**: Es el protocolo principal utilizado para consultas DNS. La mayoría de las consultas utilizan **UDP** debido a su baja latencia y el hecho de que la mayoría de las respuestas DNS son pequeñas.
  - **TCP/53**: Se utiliza cuando las respuestas exceden los **512 bytes** o para operaciones especiales como las **transferencias de zona** entre servidores DNS (AXFR).

## Protocolos de Transporte Utilizados por DNS

- **UDP (User Datagram Protocol)**: DNS utiliza **UDP** para realizar consultas normales debido a que es rápido y no requiere una conexión establecida. UDP es ideal para la mayoría de las consultas, ya que la mayoría de las respuestas DNS son pequeñas y se ajustan al límite de 512 bytes.

- **TCP (Transmission Control Protocol)**: Si una respuesta DNS es demasiado grande para enviarse por UDP o si se realiza una **transferencia de zona** entre un servidor maestro y un esclavo, el DNS cambia al **TCP**. El uso de TCP garantiza la fiabilidad y la entrega completa de la información.

## Tipos de Servidores DNS

1. **Servidor Recursivo**: Recibe la consulta del cliente y hace todo el trabajo necesario para encontrar la respuesta, preguntando a otros servidores si es necesario.
2. **Servidor Raíz**: Es el punto de partida para la resolución de nombres, proporcionando referencias a los servidores TLD.
3. **Servidor TLD**: Gestiona los dominios de nivel superior como `.com`, `.org`, etc.
4. **Servidor Autoritativo**: Tiene la respuesta definitiva para una zona específica, como los registros A de un dominio.

## Configuracion de DNS maestro, DNS esclavo y DDNS

- [DNS maestro](doc/dns_master.md)

- [DNS esclavo](doc/dns_esclavo.md)

- [DDNS](doc/ddns.md)

## Diagrama de la Topología del DNS

A continuación se muestra un diagrama que representa la topología del sistema DNS y cómo se realiza la resolución de nombres:

```
+--------------------+
|    Cliente (PC)    |
+--------+-----------+
         |
         v
+--------+-----------+
| Servidor Recursivo |
+--------+-----------+
         |
         v
+--------+----------+
|   Servidor Raíz   |
+--------+----------+
         |
         v
+--------+-----------+
|    Servidor TLD    |
| (.com, .org, etc.) |
+--------+-----------+
         |
         v
+--------+--------------+
| Servidor Autoritativo |
|     (vinylo.org)      |
+-----------------------+
```

- **Cliente**: Dispositivo que inicia la consulta DNS.
- **Servidor Recursivo**: Resuelve el nombre, contactando a otros servidores si es necesario.
- **Servidor Raíz**: El punto de inicio para todas las consultas DNS.
- **Servidor TLD**: Gestiona los dominios de nivel superior.
- **Servidor Autoritativo**: Proporciona la respuesta definitiva sobre un dominio específico.
