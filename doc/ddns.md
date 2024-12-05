# Configuración de un DDNS con BIND9 y ISC-DHCP-Server

Este documento describe cómo configurar un sistema de DNS dinámico (DDNS) utilizando BIND9 y ISC-DHCP-Server en un entorno Linux. La configuración de DDNS permite que los clientes DHCP actualicen automáticamente el servidor DNS, asegurando que los nombres de dominio estén siempre sincronizados con sus respectivas direcciones IP.

## Requisitos

- Servidor Linux con permisos de administrador (root).
- Instalación de BIND9.
- Instalación de ISC-DHCP-Server.
- Conocimientos básicos sobre configuración de redes y DNS.

## Paso 1: Instalación de BIND9 y ISC-DHCP-Server

Instala los paquetes necesarios:

```bash
sudo apt update
sudo apt install bind9 isc-dhcp-server
```

## Paso 2: Configuración de BIND9

### Crear la Clave TSIG

Para permitir que el servidor DHCP actualice la zona DNS, es necesario usar una clave TSIG. Crea la clave con el comando `dnssec-keygen`:

```bash
cd /etc/bind
sudo dnssec-keygen -a HMAC-SHA256 -b 256 -n USER dhcpupdate
```

Esto generará dos archivos (`.key` y `.private`). Copia la clave del archivo `.key` y agrégala al archivo `/etc/bind/named.conf.local`:

```bash
key "dhcpupdate" {
    algorithm HMAC-SHA256;
    secret "clave_generada";
};
```

## Paso 3: Configuración de ISC-DHCP-Server

Edita el archivo de configuración de DHCP ubicado en `/etc/dhcp/dhcpd.conf` para incluir la configuración de DDNS:

```bash
# Clave TSIG para la actualización DDNS
key dhcpupdate {
    algorithm HMAC-SHA256;
    secret "clave_generada";
}

# Configuración de DDNS
ddns-updates on;
ddns-update-style interim;

# Configurar la zona para actualización
ddns-domainname "example.com.";
ddns-rev-domainname "in-addr.arpa.";

# Servidor DNS
dns-update-key /etc/bind/dhcpupdate.key;

# Subred configurada
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option domain-name "example.com";
    option domain-name-servers 192.168.1.1;
    option routers 192.168.1.1;
}
```

## Paso 4: Permisos y Seguridad

- Asegúrate de que los archivos de clave TSIG tengan los permisos adecuados para evitar accesos no autorizados:

```bash
sudo chown root:bind /etc/bind/Kdhcpupdate.*
sudo chmod 640 /etc/bind/Kdhcpupdate.*
```

- Edita el archivo `/etc/bind/named.conf.options` para permitir consultas y definir las interfaces en las que escuchará el servidor.

## Paso 5: Reiniciar los Servicios

Reinicia ambos servicios para aplicar los cambios:

```bash
sudo systemctl restart bind9
sudo systemctl restart isc-dhcp-server
```

## Verificación

Para comprobar que las actualizaciones se están realizando correctamente, puedes revisar los archivos de registro de BIND y DHCP:

```bash
sudo tail -f /var/log/syslog
```

Asegúrate de que no haya errores y que las entradas DNS se estén actualizando como se espera.
