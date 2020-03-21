# Práctica 2 SWAP

En esta práctica se trata de aprender a clonar información entre nuestras dos máquinas y realizar una sincronización automática de los contenidos de /var/www entre ambas.

## 1. Copia de un directorio

Como se apunta en el guión de la práctica, existen varias formas de copiar archivos de una máquina a otra. En concreto se apuntan dos: una mediante SSH y otra mediante SCP, la cual hace copias seguras y encriptadas de los archivos que deseemos usando SSH.

Debido a la simplicidad de su uso, priorizaremos el uso de SCP. Sin embargo usaremos ambos métodos en esta práctica, dado que se exponen ambos.

### 1.1. tar + SSH

Este método tiene como ventaja que es posivle copiar la compresión de un directorio sin necesidad de crear el archivo comprimido en el equipo emisor, aspecto útil en caso de tener elevadas restricciones de almacenamiento en nuestro servidor. Para ello usaremos el siguiente comando:

~~~
tar -czvf - directorio | ssh usuario@equiporemoto 'cat > ~/archivo.tgz’
~~~

En nuestro caso, *directorio* será /var/www/, *archivo* será copiaSSH y equiporemoto será "192.168.56.102", es decir, nuestra M2.

![Envío copia SSH](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica2/enviocopiassh.png)

Comprobamos en la máquina destino que el archivo ha llegado correctamente.

![Recepción copia SSH](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica2/recepcioncopiassh.png)

Y por último extraemos el contenido del archivo comprimido en la ruta correspondiente con el siguiente comando:

~~~
tar -xzvf archivo.tgz -C /
~~~

Siendo nuestro archivo el mismo que hemos traído desde M1. Aplicando el comando y comprobando que todo se ha ejecutado correctamente vemos lo siguiente:

![Extracción copia SSH](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica2/extraccioncopiassh.png)

Para comprobar que se ha copiado correctamente nos fijamos en la fecha de última modificación que nos aparece al ejecutar ls sobre el directorio */var/www/*. Además, hemos borrado el archivador una vez hemos comprobado que todo se ha realizado correctamente.

### SCP

Este método, como hemos adelantado, permite realizar copias seguras y encriptadas de los archivos y directorios que se quiera transferir de un equipo a otro a través de SSH.

Esta alternativa nos ofrece igual protección ante ataques que la anterior, pero requiere menor uso de instrucciones que el anterior método, ya que con un solo comando realizamos exactamente la misma labor que con la serie de comandos y comprobaciones que usamos en el anterior apartado.

Este comando es el siguiente:

~~~
scp -r directorio usuario@equiporemoto:/directorio
~~~

En nuestro caso, el comando que hemos de usar es el siguiente:

~~~
sudo scp -r /var/www/ adizqpoz@192.168.56.102:/var
~~~

Y la salida que nos proporciona es la siguiente:

![Intento copia SCP](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica2/intentocopiascp.png)

Como vemos, el usuario de la máquina M2 no es capaz de copiar la carpeta de la máquina 1 porque no tiene los permisos suficientes para llevar esa labor a cabo. 

Se podría conectar con el usuario root de la otra máquina. Sin embargo eso sería muy peligroso, e incluso debería estar deshabilitado en la configuración de SSH. Sin embargo, esa labor no la abordaremos por el momento. 

La otra opción, mucho más viable, sería otorgar la propiedad del directorio donde alojamos nuestro sitio web al usuario al cual podemos conectar por SSH. Esto se hace con la siguiente orden:

~~~
sudo chown adizqpoz:adizqpoz -R /var/www/
~~~

Una vez hecho esto, intentamos de nuevo ejecutar el comando anterior y obtenemos esto como resultado:

![Envío copia SCP](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica2/enviocopiascp.png)

Observamos que la transfecencia se realiza con éxito. Sin embargo, es buena práctica verificar que nuestro objetivo se ha cumplido. Por ello, comprobamos en la máquina receptora guiándonos por la fecha de la última actualización.

![Recepción copia SCP](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica2/recepcioncopiascp.png)

Como suponíamos tras la anterior imagen, la copia de nuestra carpeta /var/www/ ha sido un éxito.

## 2. Clonar directorios con rsync

A pesar de que aparentemente funcionan bien los dos métodos anteriormente utilizados, para grandes cantidades de información, y para filtrar qué archivos deseamos que se clonen y qué archivos no, una herramienta más óptima para ello es *rsync*.

Se indica cómo instalar el programa en nuestros servidores. Sin embargo, ya lo tenemos instalado, así que esa labor ya no es necesaria.

También se nos indica cambiar el propietario del directorio principal del servidor web, lo cual hemos hecho en el paso anterior para poder realizar correctamente esas tareas, así que lo que nos queda en esta sección es ejecutar el comando necesario para ejecutar nuestra tarea, el cual es el siguiente:

~~~
rsync -avz -e ssh 192.168.56.101:/var/www/ /var/www/
~~~

Este comando realiza una transferencia que respeta fielmente la estructura del directorio fuente, incluyendo enlaces simbólicos, y es comprimido para realizar la transferencia de forma más eficiente. Su resultado y comprobación es el siguiente:

![Copia rsync](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica2/copiarsync.png)

Como observamos, todo se ha realizado correctamente.

Aunque no se haga una demostración de esto debido a la simplicidad de nuestro servidor web, merece la pena reseñar una funcionalidad de este programa que se apunta en el guión de prácticas de esta práctica.

Esta funcionalidad es la de omitir la copia de ciertos archivos o subdirectorios. Es útil para no realizar la copia de logs del servidor web, que son datos individuales de cada servidor, y que deben ser revisados por separado. El ejemplo de comando que se expone es el siguiente:

~~~
rsync -avz --delete --exclude=\*\*/stats --exclude=\*\*/error --exclude=\*\*/files/pictures -e ssh maquina1:/var/www/ /var/www/
~~~

Siempre que se desee alguna otra funcionalidad siempre se puede consultar el manual.

## 3. Configuración de SSH

En realidad, nuestra intención como administradores de este sistema no debería consistir en realizar una copia de un equipo a otro de forma puntual, sino que un script sea capaz de realizar copias periódicas de un equipo a otro. 

Para ello necesitamos que ambos equipos puedan tener acceso sin necesidad de introducir contraseña. La solución para ello pasa por habilitar un mecanismo de clave pública/privada, de forma que la clave pública sirva como contraseña para el equipo que debe acceder al dueño de la clave privada pueda hacerlo sin necesidad de una contraseña.

Para ello utilizamos el siguiente comando:

~~~
ssh-keygen -b 4096 -t rsa
~~~

El método utilizado para el cifrado de las claves será el método RSA, ya que el RSA1 utiliza un protocolo de SSH anterior, y por tanto, menos seguro, y el método DSA está orientado principalmente a la firma electrónica de documentos, y no puede generar pares de claves de tamaño mayor a 1024 bits.

El resultado del uso de este comando es el siguiente:

![Generar claves](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica2/generapubpriv.png)

Posteriormente, copiaremos la clave pública con un comando específico para esta labor, que simplifica significativamente la ejecución de la orden respecto a los métodos de copia de archivos anteriormente mostrados en este documento. El comando mencionado es el siguiente:

~~~

~~~

Al ejecutarlo, obtenemos esta salida:

![Copiar clave pública](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica2/copiaclavepublica.png)

Y para comprobar que hemos hecho esta serie de acciones correctamente, intentaremos conectar con el servidor M1.

![Acceso sin clave](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica2/accesosinclave.png)

Como vemos, podemos acceder con éxito al servidor principal mediante SSH sin uso de contraseña. También se podría, de forma adicional, revisar la configuración de SSH para, por ejemplo, asegurarnos de que no se puede acceder al equipo identificándose como *root* o inhabilitar el acceso mediante contraseña para que únicamente puedan acceder los equipos con los que se tenga un acceso mediante clave pública/privada, por ejemplo.

Para ello accedemos al archivo */etc/ssh/sshd_config* y comprobamos los campos pertinentes a los aspectos que acabamos de señalar.

![Acceso a root](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica2/confsshroot.png)

Como podemos observar, por defecto ya se impide el acceso al root mediante contraseña, pero sí que se podría mediante clave pública/privada. Para cambiar esto, debemos descomentar la línea y asignar a esa variable el valor "no".

![Acceso por contraseña](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica2/confsshclave.png)

Por otro lado, como ya sabíamos, se puede acceder a M1 mediante contraseña, y esto también puede deshabilitarse de la misma manera que en la opción anterior. En caso de que necesitemos que obtenga M1 una nueva clave pública de otro servidor en el futuro, debemos volver a habilitar esa opción. Por ese motivo, por el momento, no vamos a cambiar esa opción. El momento adecuando sería cuando el sistema esté completo, a falta de las posibles mejoras. Cuando se necesite conectar alguna máquina nueva, volveremos a habilitar el acceso mediante contraseña, y una vez transferida la clave pública, se volvería a deshabilitar el acceso al servidor mediante contraseña.

Cada vez que realicemos un cambio en este archivo debemos reiniciar ssh mediante la siguiente orden:

~~~
service sshd restart
~~~

## 4. Automatización de tareas

El motivo por el que habilitamos un sistema de claves pública/privada es porque el servidor puede encargarse por sí solo de mantener su información actualizada, y si hemos de introducir una contraseña cada vez que el sistema se actualice, la automatización de esta acción es baja.

Por ello, usaremos el demonio *cron*, cuyo cometido es lanzar procesos automáticamente, sea en un momento determinado, o periódicamente.

Para ello, debemos modificar el archivo /etc/crontab/ agregando una nueva línea al mismo. Como lo que deseamos es que cada hora se ejecute el comando *rsync -avz 192.168.56.101:/var/www/ /var/www/*, para hacer una copia desde M1 hasta M2, insertaremos la siguiente línea:

![Modificación de /etc/crontab](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica2/crontab.png)

Esperamos hasta que comience la siguiente hora y comprobamos que el comando se ha ejecutado al comenzar la hora buscando al final del archivo */var/log/syslog*, que contiene los logs del sistema.

![Comprobación de sincronización automática](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica2/compruebacron.png)

Como podemos observar, el comando se ha ejecutado correctamente. Con esto ya hemos cumplido con el objetivo de esta práctica, poder sincronizar periódicamente de manera automática el contenido del servidor web M2 con el de M1.

***

Autor: Adrián Izquierdo Pozo

Si desea ver el archivo Markdown puede verlo [en mi repositorio de Github](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica2/practica2.md)
