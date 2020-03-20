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

Aunque no se haga una demostración de esto debido a la simplicidad de nuestro servidor web, merece la pena reseñar una funcionalidad de este programa que se apunta en [el guión de prácticas de esta práctica](https://pradogrado1920.ugr.es/pluginfile.php/441123/mod_resource/content/1/P2_guion.pdf).

Esta funcionalidad es la de omitir la copia de ciertos archivos o subdirectorios. Es útil para no realizar la copia de logs del servidor web, que son datos individuales de cada servidor, y que deben ser revisados por separado. El ejemplo de comando que se expone es el siguiente:

~~~
rsync -avz --delete --exclude=\*\*/stats --exclude=\*\*/error --exclude=\*\*/files/pictures -e ssh maquina1:/var/www/ /var/www/
~~~

Siempre que se desee alguna otra funcionalidad siempre se puede consultar el manual.

***

Autor: Adrián Izquierdo Pozo

Si desea ver el archivo Markdown puede verlo [en mi repositorio de Github](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica2/practica2.md)
