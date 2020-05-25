# Práctica 6 SWAP

El objetivo de esta práctica es crear un servidor NFS en el que se puedan almacenar y compartir datos. El fin último de esto es que podamos tener las bases de datos de la granja web en nuestro servidor NFS, y que los servidores web se comuniquen con el mismo para recibir los datos.

Al acabar esta práctica sabemos tener servidores dedicados al balanceo de carga, servidores dedicados a servir los sitios web y servidores que almacenan las bases de datos que utilizan nuestros sitios web, y todo ello correctamente configurado. Con estos conocimientos somos capaces de crear una granja web de forma escalable, ya que si deseamos mayor capacidad, simplemente debemos adquirir y crear más máquinas, además de añadirlas a las distintas configuraciones de seguridad o de replicado de datos.

## 1. Creación del servidor NFS

Para crear nuestro servidor NFS debemos crear una nueva máquina virtual al igual que creamos las otras máquinas, con un adaptador NAT y un adaptador sólo-anfitrión, el cual tiene una dirección IP estática, que en este caso será la 192.168.56.104.

Posteriormente instalaremos el servicio NFS, tanto para servidor como para cliente, y rpcbind, dado que es necesario para que los servidores web puedan acceder de forma remota.

Posteriormente lo que hacemos es crear el directorio que tenemos pensado compartir a nuestros servidores web. En este caso serán simples archivos de texto, pero se puede aplicar a estructuras de datos más complejas de ser necesario.

![Creación directorio compartido](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica6/dircompartido.png)

Ahora damos permiso para que los dos servidores web puedan acceder a nuestra carpeta compartida, y al reiniciar el servicio todo debe estar correcto.

![NFS ok](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica6/nfsservicestatus.png)

## 2. Configuración de los clientes

En este apartado lo que debemos hacer es configurar a los servidores web para que puedan acceder a nuestro servidor nfs mediante una carpeta compartida. 

Para ello, instalamos el cliente NFS y rpcbind. Posteriormente, en una ruta cualquiera, en nuestro caso, el home de nuestro usuario, creamos una carpeta *datos*, la cual será la carpeta compartida. Además, damos permiso a todos los usuarios.

Posteriormente, montamos la carpeta exportada por el servidor en el cliente, y comprobamos que todo funciona correctamente.

![Prueba carpeta compartida](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica6/pruebacarpetacompartida.png)

Sin embargo, cuando montamos la carpeta no se trata de un cambio permanente en el sistema. Por tanto, debemos añadir una línea en el archivo /etc/fstab, el cual controla el sistema de montaje de dispositivos, para montar la carpeta compartida por el servidor NFS.

![Configuración para montar NFS automátcamente](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica6/confnfsauto.png)

Tras ello ya no hemos de preocuparnos por montar manualmente la carpeta compartida cada vez que arranquemos el sistema.

### 3. Seguridad en el servidor NFS

En este apartado asumimos que los servidores web no tienen el cortafuegos habilitado. En un sistema real, cada uno de los cortafuegos de los servidores web deben poder enviar y recibir peticiones por los puertos que se utilizan para hacer posible el sistema NFS, los cuales enumeraremos posteriormente.

En primer lugar, utilizamos las reglas necesarias para denegar todo tráfico entrante, y aceptar las conexiones establecidas y relacionadas.

Posteriormente debemos tener en cuenta los servicios que se utilizan:

- nfs: utiliza el puerto 2049 para tcp y udp.
- portmapper: utiliza el puerto 111 para tcp y udp
- mountd: por defecto utiliza puertos dinámicos. Para fijar un puerto para este servicio debemos modificar una línea en el archivo /etc/defaults/nfs-kernel-server, para que escuche específicamente el puerto 2000, por ejemplo. Es decir, debemos tener una línea tal que así:

~~~
RPCMOUNTDOPTS="--manage-gids -p 2000"
~~~

![Configuración para fijar el puerto mountd](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica6/confmountd.png)

- nlockmgr: es parte de un módulo del kernel. Debemos crear un archivo de configuracón en la carpeta /etc/sysctl.d/ donde habilitaremos para este servicio el puerto 2001 para tcp y 2002 para udp:

~~~
fs.nfs.nlm_tcpport = 2001
fs.nfs.nlm_udpport = 2002
~~~

Posteriormente lanzamos el nuevo archivo de configuración, y reiniciamos el servidor NFS.

Ahora será útil comprobar que utilizamos los puertos que hemos fijado con el comando rpcinfo -p server:

![Puertos fijados para los servicios de NFS](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica6/portservicesnfs.png)

Sabiendo esto, ya podemos configurar correctamente el tráfico que puede entrar, no salir. Determinamos de dónde puede venir el tráfico TCP y UDP y sus puertos con las dos últimas reglas de este script:

![Firewall de NFS](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica6/conffirewallm4.png)

Este script, tal como hicimos en la práctica 4, lo ejecutaremos en cada arranque con el demonio cron.

Tras reiniciar, realizamos la prueba de que todo funciona correctamente creando un archivo en la carpeta compartida desde M4:

![Prueba de NFS tras el firewall](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica6/pruebafirewallnfs.png)

***

Autor: Adrián Izquierdo Pozo

Si desea ver el archivo Markdown puede verlo [en mi repositorio de Github](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica6/practica6.md)
