# Práctica 3 SWAP

En esta práctica se trata de aprender a balancear la carga entre nuestros dos servidores m1 y m2 mediante el uso de una tercera máquina virtual m3 con varios balanceadores software.

## 1. NGINX

El primer balanceador que vamos a probar es NGINX. Este software es en realidad un servidor web ligero utilizado por varios sitios web de renombre. Además, tiene la funcionalidad de balanceador de carga, la cual es una de las funcionalidades más utilizadas para este software.

### 1.1. Configuración

Para configurar esta herramienta como balanceador de carga, debemos crear un archivo en */etc/nginx/conf.d/default.conf*, dado que no existe en nuestro servidor tras su instalación.

En este archivo definimos un *upstream*, grupo al cual se va a redirigir el tráfico que reciba nuestro servidor web. En él se introducen las IP de nuestros servidores.

A continuación, definimos el código del servidor, es decir, nuestro balanceador. Debemos hacer que escuche el puerto 80, ponerle un nombre al servidor, definir los archivos de log, el root, el protocolo HTTP y la redirección de  tráfico.

![Configuración nginx](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica3/nginxconf.png)

Si quisiéramos definir un reparto ponderado, debemos definir para cada servidor un parámetro *weight*, igual al peso que le queramos dar, con una línea del estilo de la siguiente:

~~~
server ip1 weight=2;
server ip2 weight=1;
~~~

Reiniciamos el servicio nginx y probamos desde nuestro navegador, dándonos el siguiente resultado:

![Prueba fallida nginx](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica3/nginxpruebabalanceofallida.png)

Este error se debe a que en el archivo de configuración de nginx situado en */etc/nginx/nginx.conf* el servidor web está configurado como tal, con lo cual no llega a redirigir el tráfico. Para ello comentamos la línea que provoca este hecho, relativa a los sitios habilitados.

Una vez hecho esto, probamos a ejecutar curl hacia m3, y vemos lo siguiente:

![Prueba nginx con éxito](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica3/nginxpruebadefinitiva.png)

Con esto, ya hemos creado un balanceador de carga sencillo con NGINX.

## 2. HAProxy

HAProxy es un balanceador de carga que asume las funcionalidades de proxy, con lo cual es capaz de redirigir cualquier tipo de tráfico.

Se trata de un balanceador de altas prestaciones, con lo cual su uso está muy indicado en caso de usar un balanceador software.

Tras instalarlo, para realizarle la configuración hay que definir el frontend, es decir, la recepción de peticiones desde el exterior, y el backend, en el que incluimos nuestros servidores, y definimos el método de reparto.

![Configuración haproxy](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica3/haproxyconf.png)

Si quisiéramos definir un reparto ponderado, debemos definir para cada servidor un parámetro *weight*, igual al peso que le queramos dar, con una línea del estilo de la siguiente:

~~~
server m1 ip1:80 maxconn 32 weight 2
server m2 ip2:80 maxconn 32 weight 1
~~~
Posteriormente, iniciamos el servicio haproxy, asegurándonos de que nginx está inhabilitado, y probamos el balanceo:

![Prueba haproxy con éxito](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica3/haproxyprueba.png)

Con esto, ya hemos creado un balanceador de carga sencillo con HAProxy, con a simple vista una configuración más intuitiva que la de NGINX.

## 3. Aplicación de benchmark a nuestra granja web.

Una vez configurados ambos balanceadores debemos poner nuestra granja web a prueba. En este caso lo haremos con la herramienta *apache benchmark*.

Para ello, vamos a ejecutar la siguiente orden para ambos casos:

~~~
ab -n 10000 -c 10 http://192.168.56.103/ejemplo.html
~~~

Para el balanceador NGINX obtenemos lo siguiente:

![Comprobación reparto de carga nginx](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica3/nginxbenchmarktop.png)

![Datos benchmark nginx](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica3/nginxbenchmarkdata.png)

Y para el balanceador HAProxy obtenemos los siguientes datos:

![Comprobación reparto de carga haproxy](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica3/haproxybenchmarktop.png)

![Datos benchmark haproxy](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica3/haproxybenchmarkdata.png)

Como vemos, al parecer HAProxy es un balanceador de carga más eficiente que NGINX, con lo cual, en principio, preferiremos el uso del primero.

***

Autor: Adrián Izquierdo Pozo

Si desea ver el archivo Markdown puede verlo [en mi repositorio de Github](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica2/practica2.md)
