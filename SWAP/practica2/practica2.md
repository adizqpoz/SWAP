# Práctica 2 SWAP

En esta práctica se trata de aprender a clonar información entre nuestras dos máquinas y realizar una sincronización automática de los contenidos de /var/www entre ambas.

## 1. Copia de un directorio

Como se apunta en el guión de la práctica, existen varias formas de copiar archivos de una máquina a otra. En concreto se apuntan dos: una mediante SSH y otra mediante SCP, la cual hace copias seguras y encriptadas de los archivos que deseemos usando SSH.

Debido a esto, y a la simplicidad de su uso, priorizaremos el uso de SCP. Sin embargo usaremos ambos métodos en esta práctica, dado que se exponen ambos.

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

***

Autor: Adrián Izquierdo Pozo

Si desea ver el archivo Markdown puede verlo [en mi repositorio de Github](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica2/practica2.md)
