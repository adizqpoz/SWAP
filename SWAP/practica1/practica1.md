# Práctica 1 SWAP

En esta práctica se trata de instalar dos servidores en dos máquinas virtuales, y hacer que estén interconectados mediante una red interna, y que tengan la pila LAMP instalada y funcionando correctamente.

## 1. Instalación de las máquinas

Para ello se harán dos instalaciones diferentes en cada máquina, desde ahora *m1* y *m2*. En *m1* antes de la instalación se hará una conexión adicional para el adaptador en modo sólo anfitrión para conectarlo ya a la red interna, y en *m2* se hará únicamente la conexión NAT a través de la cual se podrá acceder a Internet a través del dipositivo anfitrión.

![Configuración MV 1](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica1/confswap1.png)
![Configuración MV 2](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica1/confswap2.png)

Posteriormente se procede a instalar las dos máquinas, configurando el idioma, el perfil, la red, la instalación de ciertos servicios...

Nosotros avanzamos en todos los pasos sin realizar cambios excepto en estos dos:

![Perfil](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica1/perfilm1.png)
![Instalación SSH](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica1/ssh.png)

Notamos que en esta imagen ya hacemos la instalación de SSH.

Hecho esto, esperamos a que se efectúe la instalación y reiniciaremos.

## 2. Instalación de la pila LAMP

Una vez hecho esto procedemos a instalar la pila LAMP en ambos servidores. Para ello se ejecuta el siguiente listado de comandos:

~~~
sudo apt install apache2
sudo apt install mysql-server mysql-client
~~~

Comprobamos que Apache funciona correctamente con el comando 

~~~
service apache2 status
~~~

Lo cual nos muestra:

![Estado Apache](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica1/apache.png)

Y con esto ya tenemos instalada la pila LAMP. Ahora debemos asegurarnos de que ambas máquinas pueden comunicarse.

## 3. Configuración de red

Para esta labor, teniendo en cuenta las dos instalaciones diferentes que hemos realizado, vamos a ver las dos perspectivas:

### Configuración para M1

En este caso la configuración está hecha de forma automática durante la instalación. Para comprobarlo usaremos la siguiente orden:

~~~
ifconfig
~~~

Y en este caso vemos que todo está correctamente configurado

![Redes M1](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica1/red1.png)

### Configuración para M2

Esta máquina aún no está conectada a la red interna con el conector sólo-anfitrión, por lo cual lo que debemos hacer es:

1. Apagar la máquina.
2. Abrir la configuración de la máquina virtual correspondiente a M2.
3. En el apartado *Red* habilitar el segundo conector de red, y seleccionamos el conector sólo-anfitrión.

Ahora procedemos a comprobar las redes a las que está conectada M2 con el comando usado en M1, y vemos lo siguiente:

![Redes 2 - Paso 1](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica1/red2.1.png)

Como se puede observar, esta máquina aún no está conectada a la red interna. Ahora debemos revisar el archivo de configuración de la red, el cual está alojado en la ruta */etc/netplan/50-cloud.init.yaml*, y vemos lo siguiente:

![Netplan - Paso 1](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica1/netplan1.png)

Como vemos, el conector *enp0s8* no está configurado. Lo configuramos del mismo modo que está configurado *enp0s3*.

![Netplan - Paso 2](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica1/netplan2.png)

Comprobamos y vemos lo siguiente:

![Redes 2 - Paso 2](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica1/red2.2.png)

Vemos que no tiene asignada la IP que le corresponde. Para cambiarla a la que deseamos debemos volver a configurar nuestro archivo de configuración, acabando de esta forma:

![Netplan - Paso 3](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica1/netplan3.png)

Al verificar observamos que todo está a priori correctamente configurado.

![Redes 2 - Paso 3](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica1/red2.3.png)

***

En condiciones normales comprobaríamos que esto funciona utilizando el comando *ping*. Sin embargo, debido a la naturaleza de esta práctica, esta comprobación la haremos en el siguiente apartado, cuya finalidad es exactamente esa.

## 4. Cuestiones a resolver

Las tareas que nuestros servidores deben ser capaces de realizar son las siguientes:

- Acceder por ssh de una máquina a otra
- Acceder mediante la herramienta curl desde una máquina a la otra

En teoría, tras el trabajo previo realizado debemos ser capaces de, de manera inmediata, resolver estas cuestiones.

### 4.1. Acceder por ssh de una máquina a otra

Para ello debemos ejecutar el siguiente comando en cada una de las máquinas:

~~~
ssh adizqpoz@<ip de la máquina complementaria>
~~~

Cuando hacemos esto en ambas máquinas vemos el siguiente resultado:

![Resultado SSH satisfactorio](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica1/res_ssh.png)

Podemos observar que el servicio ssh funciona como debe.

### 4.2. Acceder mediante la herramienta curl desde una máquina a la otra

Para ello, en la ruta */var/www/html/* creamos un archivo "ejemplo.html" con el contenido que se nos da como ejemplo en [el guión de la práctica](https://pradogrado1920.ugr.es/pluginfile.php/441121/mod_resource/content/1/P1_guion.pdf), es decir:

~~~
<HTML>
    <BODY>
        Web de ejemplo de adizqpoz para SWAP
    </BODY>
</HTML>
~~~

Posteriormente usamos el siguiente comando en cada una de las máquinas:

~~~
curl <ip de la máquina complementaria>/ejemplo.html
~~~

Ejecutando este comando observamos lo siguiente:

![Resultado curl satisfactorio](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica1/res_curl.png)

Por tanto, el servidor Apache funciona correctamente y podemos dar por concluida esta práctica.

***

Autor: Adrián Izquierdo Pozo
