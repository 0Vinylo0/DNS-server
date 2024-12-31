# Configurar un Servidor DNS Maestro con BIND

Este tutorial te guiará en la configuración de un servidor DNS maestro utilizando **BIND**. Vamos a configurar un servidor DNS que sea autoritativo para una zona y permita la administración y la resolución de nombres dentro de un dominio.

## Requisitos Previos
- Sistema operativo basado en Linux (Debian, Ubuntu, CentOS, etc.).
- Privilegios de **sudo**.
- Paquete **BIND9** instalado:
  ```bash
  sudo apt-get update
  sudo apt-get install bind9 bind9utils bind9-doc
  ```

## Paso 1: Configurar el Archivo de Zona

1. **Editar la Configuración de la Zona**:
   
   Abre el archivo de configuración de BIND para definir una nueva zona:
   ```bash
   sudo nano /etc/bind/named.conf.local
   ```

2. **Agregar la Zona Maestro**:

   Agrega la siguiente configuración para definir la zona:
   ```plaintext
   zone "vinylo.org" {
       type master;
       file "/etc/bind/db.vinylo.org";
       allow-transfer { 192.168.1.28; };  // IP del servidor esclavo
       also-notify { 192.168.1.28; };     // Notificar al esclavo cuando haya cambios
   };

   zone "0.168.192.in-addr.arpa" {
       type master;
       file "/etc/bind/db.192.168.1.rev";
       allow-transfer { 192.168.1.28; };  // IP del servidor esclavo
       also-notify { 192.168.1.28; };     // Notificar al esclavo cuando haya cambios
   };
   ```
   - **`type master`**: Define este servidor como el **maestro** para la zona `vinylo.org` y su zona inversa.
   - **`file "/etc/bind/db.vinylo.org"`**: Archivo donde se almacena la configuración de la zona.
   - **`file "/etc/bind/db.192.168.1.rev"`**: Archivo donde se almacena la configuración de la zona inversa.
   - **`allow-transfer`**: Permite la transferencia de zona al servidor esclavo.

## Paso 2: Crear el Archivo de Zona

1. **Crear el Archivo de Zona**:

   Crea un archivo de zona para `vinylo.org`:
   ```bash
   sudo nano /etc/bind/db.vinylo.org
   ```

2. **Agregar la Configuración de la Zona**:

   Utiliza el siguiente contenido como plantilla para el archivo de zona:
   ```plaintext
   $TTL    604800
   $ORIGIN vinylo.org.
   @       IN      SOA     ns1.vinylo.org. root.vinylo.org. (
                              2023120101 ; Serial (formato AAAAMMDDNN)
                              604800     ; Refresh
                              86400      ; Retry
                             2419200    ; Expire
                              604800 )   ; Negative Cache TTL
   ;
   @        IN      NS      ns1.vinylo.org.
            IN      NS      ns2.vinylo.org.
    
   ; Direcciones IP principales
   @       IN      A       192.168.1.14
   ns1     IN      A       192.168.1.14
   ns2     IN      A       192.168.1.28
   www     IN      A       192.168.1.13
    
   ; Servidor de correo
   @       IN      MX 10  mail.vinylo.org.
   mail    IN      A       192.168.1.15
    
   ; Servidor FTP
   ftp     IN      A       192.168.1.16
    
   ; Servidor de chat
   chat    IN      A       192.168.1.17
    
   ; Alias (CNAME) para simplificar accesos
   webmail IN      CNAME   mail.vinylo.org.
   docs    IN      CNAME   www.vinylo.org.
   ```
   - **`$ORIGIN`**: Define el punto de partida o base del nombre de dominio en un archivo de zona DNS.
   - **`$TTL`**: Define el tiempo de vida de los registros en la caché.
   - **`SOA`**: Define el servidor de autoridad y los parámetros de la zona.
   - **`NS`**: Define los servidores de nombres de la zona.

## Paso 3: Crear el Archivo de Zona Inversa

1. **Crear el Archivo de Zona Inversa**:

   Crea un archivo de zona inversa para la red `192.168.1.0/24`:
   ```bash
   sudo nano /etc/bind/db.192.168.1.rev
   ```

2. **Agregar la Configuración de la Zona Inversa**:

   Utiliza el siguiente contenido como plantilla para el archivo de zona inversa:
   ```plaintext
   $TTL    604800
    $ORIGIN 1.168.192.in-addr.arpa.
    @       IN      SOA     ns1.vinylo.org. root.vinylo.org. (
                              2023120101 ; Serial (formato AAAAMMDDNN)
                              604800     ; Refresh
                              86400      ; Retry
                              2419200    ; Expire
                              604800 )   ; Negative Cache TTL
    ;
            IN      NS      ns1.vinylo.org.
            IN      NS      ns2.vinylo.org.
    
    ; Registros PTR
    14      IN      PTR     ns1.vinylo.org.      ; Dirección principal del servidor
    28      IN      PTR     ns2.vinylo.org.      ; Servidor secundario
    13      IN      PTR     www.vinylo.org.      ; Página web
    15      IN      PTR     mail.vinylo.org.     ; Servidor de correo
    16      IN      PTR     ftp.vinylo.org.      ; Servidor FTP
    17      IN      PTR     chat.vinylo.org.     ; Servidor de chat
   ```
   - **`$ORIGIN`**: Define el punto de partida o base del nombre de dominio en un archivo de zona DNS.
   - **`$TTL`**: Define el tiempo de vida de los registros en la caché.
   - **`SOA`**: Define el servidor de autoridad y los parámetros de la zona inversa.
   - **`NS`**: Define los servidores de nombres de la zona.
   - **`PTR`**: Define los registros inversos para mapear direcciones IP a nombres de dominio.

## Paso 4: Verificar la Configuración

1. **Verificar el Archivo de Configuración**:
   ```bash
   sudo named-checkconf
   ```
   Si no hay errores, el comando no devolverá nada.

2. **Verificar el Archivo de Zona Directa**:
   ```bash
   sudo named-checkzone vinylo.org /etc/bind/db.vinylo.org
   ```
   Esto te ayudará a detectar errores en el archivo de zona.

3. **Verificar el Archivo de Zona Inversa**:
   ```bash
   sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/db.192.168.1.rev
   ```
   Esto te ayudará a detectar errores en el archivo de zona inversa.

## Paso 5: Reiniciar el Servicio BIND

Finalmente, reinicia el servicio **BIND** para aplicar los cambios:
```bash
sudo systemctl restart bind9
```

## Paso 6: Probar la Resolución

Usa el comando **dig** para probar la resolución del dominio:
```bash
dig @localhost vinylo.org
```

Y para probar la resolución inversa:
```bash
dig @localhost -x 192.168.1.14
```

Si todo está bien configurado, deberías ver los registros correspondientes del dominio `vinylo.org` y la resolución inversa de las direcciones IP.

---

Ahora que el servidor maestro está listo, puedes continuar configurando el **servidor esclavo** para tener redundancia y alta disponibilidad.
