# Configurar un Servidor DNS Esclavo con BIND (con Zona Inversa)

Este tutorial te guiará para configurar un servidor **DNS esclavo** que pueda sincronizarse con el **servidor maestro** previamente configurado, incluyendo la zona inversa. Un servidor esclavo proporciona **redundancia** y **alta disponibilidad**, lo cual es esencial en entornos de producción.

## Requisitos Previos
- Tener un servidor **DNS maestro** ya configurado (ver tutorial "Configurar un DNS Maestro con BIND").
- Paquete **BIND9** instalado en el servidor esclavo:
  ```bash
  sudo apt-get update
  sudo apt-get install bind9 bind9utils bind9-doc
  ```
- Conectividad entre el servidor maestro y el servidor esclavo.

## Paso 1: Configurar la Zona en el Servidor Esclavo

1. **Editar el Archivo de Configuración**:

   Abre el archivo **`/etc/bind/named.conf.local`** para agregar la configuración de la zona esclava:
   ```bash
   sudo nano /etc/bind/named.conf.local
   ```

2. **Definir la Zona como Esclava**:

   Agrega la configuración para la zona que será replicada desde el maestro:
   ```plaintext
   zone "vinylo.org" {
       type slave;
       file "/var/cache/bind/db.vinylo.org";
       masters { 192.168.1.14; };  // Dirección IP del servidor maestro
   };

   zone "0.168.192.in-addr.arpa" {
       type slave;
       file "/var/cache/bind/db.192.168.0.rev";
       masters { 192.168.1.14; };  // Dirección IP del servidor maestro
   };
   ```
   - **`type slave`**: Define esta zona como esclava.
   - **`file "/var/cache/bind/db.vinylo.org"`**: Archivo donde se guardará la información replicada de la zona directa.
   - **`file "/var/cache/bind/db.192.168.0.rev"`**: Archivo donde se guardará la información replicada de la zona inversa.
   - **`masters { 192.168.1.14; };`**: Dirección del servidor maestro desde el cual se replicará la información.

## Paso 2: Verificar y Reiniciar el Servidor DNS

1. **Verificar la Configuración**:

   Antes de reiniciar el servidor, asegúrate de que la configuración sea válida:
   ```bash
   sudo named-checkconf
   ```

2. **Reiniciar BIND**:

   Reinicia el servicio **BIND** para aplicar la nueva configuración:
   ```bash
   sudo systemctl restart bind9
   ```

## Paso 3: Verificar la Transferencia de Zona

1. **Revisar los Logs**:

   Puedes revisar los logs para asegurarte de que la transferencia de zona se haya realizado correctamente. Los logs suelen encontrarse en **`/var/log/syslog`**:
   ```bash
   tail -f /var/log/syslog
   ```
   - Busca mensajes que indiquen que la **zona vinylo.org** y la **zona inversa** fueron **transferidas exitosamente** desde el maestro.

2. **Probar la Resolución**:

   Utiliza el comando **dig** para probar si el esclavo está resolviendo correctamente el dominio:
   ```bash
   dig @localhost vinylo.org
   ```
   - Para probar la resolución inversa:
   ```bash
   dig @localhost -x 192.168.1.14
   ```
   - Si el servidor esclavo está correctamente configurado, deberías obtener una respuesta con los registros de la zona y la resolución inversa.

## Paso 4: Mantener la Sincronización

El servidor **esclavo** se mantendrá sincronizado con el **maestro** automáticamente. Cuando se realicen cambios en el servidor maestro (y se incremente el **serial** en el archivo de zona), el servidor esclavo será notificado y obtendrá la versión más actualizada de la zona.

---

Con este servidor DNS esclavo, ya tienes una configuración redundante para asegurar la disponibilidad y la fiabilidad de los registros DNS, incluyendo tanto la zona directa como la zona inversa. Puedes continuar configurando **DDNS** con un servidor DHCP para automatizar las actualizaciones.
