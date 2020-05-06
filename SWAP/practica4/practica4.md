# Práctica 4 SWAP

En esta práctica hay dos objetivos principales: 

	- Aprender a servir páginas web con el protocolo HTTPS mediante el uso de SSL.
	- Aprender a configurar el firewall de un servidor web correctamente

## 1. HTTPS

En primer lugar vamos a configurar HTTPS para uno de los servidores finales, en este caso M1. Posteriormente copiaremos el certificado y la clave a M2 y a M3, con lo cual nuestra granja web debe ser capaz de servir una página web con protocolo HTTPS de forma satisfactoria.

El protocolo HTTPS, en esencia, es una solución para los problemas de seguridad que presentaba HTTP, que son los siguientes:

	- Falta de confidencialidad, ya que los datos se enviaban sin ningún tipo de cifrado, lo cual hace visible el contenido de la petición a personas no autorizadas.
	- No garantiza que la página web tenga el propietario que dice tener, ya que no se trata de un documeto firmado.
	
HTTPS mitiga estos dos problemas, dado que se genera una firma digital para el sitio web y se genera un cifrado para los datos que se envían a través de la capa de transporte.

### 1.1. Configuración

Para configurar SSL debemos en primer lugar habilitar en el servidor Apache el protocolo SSL, que nos permitirá firmar y cifrar nuestro sitio web. Para ello utilizamos el siguiente comando:

~~~
sudo a2enmod ssl
~~~

Y posteriormente reiniciamos Apache para hacer efectivos los cambios.

![Configuración Apache SSL](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica4/a2enmodcorrecto.png)

A continuación creamos nuestra clave y certificado para nuestro sitio web. Sin embargo, los navegadores detectarán nuestra firma como insegura, dado que no la ha validado ninguna entidad certificadora. Aún así, por motivos didácticos, crearemos nuestro propio certificado digital con OpenSSL.

Para ello, en el directorio /etc/apache2 debemos crear un nuevo subdirectorio llamado ssl, en el cual guardaremos la clave y certificado que se generarán. Posteriormente utilizaremos el siguiente comando para generar la clave y certificado:

~~~
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt
~~~

![Firma SSL](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica4/firmassl.png)

Tras rellenar el formulario con los datos pertinentes configuramos Apache para indicarle dónde están el certificado y la clave que se han de utilizar para servir una página HTTPS, reiniciamos Apache y comprobamos que podemos servir una de nuestras páginas mediante el protocolo citado:

![Certificado](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica4/certificado.png)

Para terminar, copiaremos mediante el comando SCP utilizado en la práctica 2 para copiar las claves de nuestro certificado a M2 y a M3, configuramos Apache en M2 de forma idéntica a M1, y configuramos M3 para que redireccione las peticiones HTTPS de igual forma que hicimos con HTTP en la práctica 3, pero indicando que utilizamos el protocolo SSL, que escuchamos el puerto 443, puerto por defecto en el que se escuchan las peticiones HTTPS, y le indicamos la ruta de los archivos recibidos por SCP.

Por último, comprobamos que nuestra infraestructura puede servir páginas HTTPS de forma satisfactoria:

![Comprobación HTTPS granja web](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica4/sshtodos.png)

## 2. iptables

iptables es una herramienta de firewall nativa de los sistemas Linux que nos permite controlar qué tipo de tráfico pueden aceptar nuestras máquinas servidoras. Esta herramienta se configura mediante una serie de reglas que se aplican a nuestro cortafuegos.

En primer lugar limpiaremos todas las reglas que tengamos en iptables para asegurarnos de que las reglas que vamos a añadir no interactúan con otras reglas que no controlamos en este momento. Para ello utilizamos las siguientes reglas:

~~~
iptables -F
iptables -X 
iptables –t nat -F
iptables –t nat -X
iptables –t mangle -F
iptables –t mangle -X 
iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT
~~~

Con ello tenemos una configuración por defecto que acepta todas las peticiones.

![Configuración por defecto iptables](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica4/iptableslimpio.png)

Posteriormente denegamos todo tráfico de información para posteriormente definir las excepciones de tráfico que permitiremos que entre y salga de nuestro servidor:

~~~
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
~~~

![Tráfico denegado iptables](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica4/iptablesbloqueatodo.png)

A continuación, para perimtir el tráfico de los servicios que utilizan nuestros servidores debemos definir un conjunto de reglas. En concreto vamos a permitir:

	- Accesos desde localhost
	- Acceso y salida de SSH desde los equipos de nuestra granja web
	- Acceso y salida de peticiones HTTP
	- Acceso y salida de peticiones HTTPS
	
Además, crearemos un script que nos permita tener el firewall configurado desde el momento en el que encendemos nuestros servidores. 

![Script configuración iptables](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica4/scriptfirewall.png)

El estado en el que queda nuestro firewall es el siguiente:

![Configuración final iptables](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica4/conffirewallfinal.png)

Por último, comprobaremos que el firewall nos permite realizar las acciones que deseamos:

![Uso SSH tras configuración de iptables](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica4/sshpostfirewall.png)

![Acceso web tras configuración de iptables](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica4/accesowebpostfirewall.png)

En este punto concluimos la práctica, a pesar de que es posible profundizar algo más en el uso del cortafuegos para toda nuestra granja web, de forma que nuestras máquinas finales sólo reciban peticiones web del balanceador, y el balanceador responda a las peticiones web, y redirijan el tráfico a las máquinas servidoras finales.

***

Autor: Adrián Izquierdo Pozo

Si desea ver el archivo Markdown puede verlo [en mi repositorio de Github](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica4/practica4.md)
