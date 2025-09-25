# Objetivo de la práctica

Configurar un firewall con Shorewall que:
- Permita acceso a Internet desde la LAN usando masquerade (NAT)
- Redirija tráfico externo al servidor web en la DMZ usando DNAT
- Bloquee todo lo demás por defecto

## Requisitos:
- Máquina Linux con Shorewall instalado
  
### Ejemplo:
```
root@lattice:~# aptitude install shorewall  shorewall-doc

Se instalarán los siguiente paquetes NUEVOS:      

  libnetfilter-cthelper0{a} shorewall shorewall-core{a} shorewall-doc 
0 paquetes actualizados, 4 nuevos instalados, 0 para eliminar y 10 sin actualizar.
Necesito descargar 7 519 kB de ficheros. Después de desempaquetar se usarán 14.8 MB.

¿Quiere continuar? [Y/n/?]
```


### Tres interfaces de red:

    eth0: hacia Internet
  
    eth1: hacia LAN (192.168.1.0/24)
  
    eth2: hacia DMZ (172.16.0.0/16)
  

## Estructura de archivos de configuración

Shorewall usa archivos en /etc/shorewall/.

El directorio de shorewall contiene los siguientes archivos después de la instalación:

conntrack  params  shorewall.conf

Es necesario copiar los archivos de ejemplo que se encuentran en

/usr/share/doc/shorewall/examples/three-interfaces/

```
Archivo      Propósito

interfaces   Define las interfaces de red

zones	     Define zonas como LAN, DMZ, net

policy	     Define políticas por defecto

rules	     Reglas específicas de tráfico

snat	     Configura NAT
```

## Configurar interfaces (interfaces)
- copiar el archivo, suponiendo que me encuentro en /etc/shorewall
  necesitas ejecutar los siguiente:
  ```
  root@lattice:/etc/shorewall# cp /usr/share/doc/shorewall/examples/three-interfaces/interfaces .
  root@lattice:/etc/shorewall# ls
  conntrack  interfaces  params  shorewall.conf
  ```

- editar el archivo con tu editor de preferencia (nano, vi, vim o emacs)
  nota: sustituir la palabra editor por el de tu preferencia
  
  editor interfaces
  
  y dentro el archivo sustituir por el nombre de la intefaz de tu equipo
  en este caso vamos a suponer algunos nombres de las interfaces
  (eth0,eth1 y eth2) y la columna de OPTIONS dejarla tal cual como viene
   en el archivo.
  ```
  #ZONE   INTERFACE       OPTIONS
  net     etho
  loc     eth1
  dmz     eth2
  ```
  

## Configurar zonas (zones)
- copiar el archivo, suponiendo que me encuentro en /etc/shorewall
  necesitas ejecutar los siguiente:
  root@lattice:/etc/shorewall# cp /usr/share/doc/shorewall/examples/three-interfaces/zones .
  root@lattice:/etc/shorewall# ls
  conntrack  interfaces  params  shorewall.conf  zones

  editor zones

  en este caso no se necesita realizar alguna modificación

## Política por defecto (policy)
- copiar el archivo, suponiendo que me encuentro en /etc/shorewall
  necesitas ejecutar los siguiente:
  ```
  root@lattice:/etc/shorewall# cp /usr/share/doc/shorewall/examples/three-interfaces/policy .
  root@lattice:/etc/shorewall# ls
  conntrack  interfaces  params  policy  shorewall.conf  zones
  ```

  editor policy

  vamos a agregar la siguiente política a nuestro archivo
  $FW     loc             ACCEPT

  y queda de la siguiente manera:
  ```
  #SOURCE DEST            POLICY          LOGLEVEL        RATE    CONNLIMIT

  loc     net             DROP
  $FW     loc             ACCEPT
  net     all             DROP            $LOG_LEVEL
  # THE FOLOWING POLICY MUST BE LAST
  all     all             REJECT          $LOG_LEVEL
  ```

## Reglas específicas (rules)
- copiar el archivo, suponiendo que me encuentro en /etc/shorewall
  necesitas ejecutar los siguiente:
  ```
  root@lattice:/etc/shorewall# cp /usr/share/doc/shorewall/examples/three-interfaces/rules .
  root@lattice:/etc/shorewall# ls
  conntrack  interfaces  params  policy  rules  shorewall.conf  zones
  ```
  editor rules
  en este caso de momento no se necesita realizar alguna modificación

## NAT (masq)
- copiar el archivo, suponiendo que me encuentro en /etc/shorewall
  necesitas ejecutar los siguiente:
  
  ```
  root@lattice:/etc/shorewall# cp /usr/share/doc/shorewall/examples/three-interfaces/snat .
  root@lattice:/etc/shorewall# ls
  conntrack  interfaces  params  policy  rules  shorewall.conf  snat  zones
  ```

  editor snat

  modificar los tipos de red (192.168.1.0 y 172.16.0.0)
  recordar que la interfaz que proporciona red es eth0, en otras palabras
  se le indica que la red tipo C y B tengan acceso a internet
  ```
  MASQUERADE            192.168.0.0/24,\
                        172.16.0.0/16  eth0
  ```

## Activar el reenvío de paquetes (IP Forwarding)

editor shorewall.conf

Busca la línea IP_FORWARDING= y cambia Keep por On, queda asi:
```
...
IP_FORWARDING=On
...
```

## Finalmente varificamos que todo este configurado adecuadamente
para esto vamos a ejecutar lo siguiente:
```
root@juan:/etc/shorewall# shorewall check

y veremos los siguiente:

Checking using Shorewall 5.2.8...
Processing /etc/shorewall/params ...
Processing /etc/shorewall/shorewall.conf...
Loading Modules...
Compiling /etc/shorewall/zones...
Compiling /etc/shorewall/interfaces...
Determining Hosts in Zones...
Locating Action Files...
Compiling /etc/shorewall/policy...
Adding Anti-smurf Rules
Adding rules for DHCP
Compiling TCP Flags filtering...
Compiling Kernel Route Filtering...
Compiling Martian Logging...
Compiling Accept Source Routing...
Compiling /etc/shorewall/snat...
Compiling MAC Filtration -- Phase 1...
Compiling /etc/shorewall/rules...
Compiling MAC Filtration -- Phase 2...
Applying Policies...
Shorewall configuration verified
```
