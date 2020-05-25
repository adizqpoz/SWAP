# Práctica 5 SWAP

El objetivo de esta práctica es, tal como hemos hecho con las páginas que servimos en nuestra granja web, tener nuestra base de datos replicada en varias máquinas virtuales. 

En este ejemplo usaremos M1 y M2, pero **no necesariamente deben estar en las mismas máquinas donde tenemos nuestras páginas web**, sino que podemos tener, por ejemplo, un cluster de máquinas en las que se alojan las bases de datos.

## 1. Creación de la base de datos

En nuestro caso, haremos una base de datos MySQL muy simple, pero que nos será útil para realizar esta práctica. Poseerá una única tabla, sin clave primaria definida.

Los pasos para crear la base de datos son los siguientes:

1. Definir el diseño de nuestra base de datos y su paso a tablas correspondiente. Este paso no está en las competencias de esta asignatura, y ya hemos decidido nuestra base de datos.

2. Entrar como administrador a la terminal de MySQL.

3. Crear la base de datos.

4. Selecionar nuestra base de datos.

5. Crear la(s) tabla(s).

6. Insertar los datos.

Podemos también en cada momento comprobar nuestras tablas, tuplas y atributos para asegurarnos de que lo hemos hecho correctamente.

![Creación base de datos 1](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica5/database1.png)

![Creación base de datos 2](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica5/database2.png)

Con esto ya tenemos nuestra base de datos preparada.

## 2. Replicado de bases de datos

Aquí, tal como hicimos para los servidores web, se expondrá cómo replicar nuestra base de datos de forma manual y de forma automática.

### 2.1. Replicado manual

Para realizar el replicado de datos de forma manual utilizaremos la herramienta mysqldump, y scp para copiar los datos replicados a la otra máquina.

Para realizar la copia, en primer lugar bloqueamos los accesos a la base de datos para así no sufrir anomalías a la hora de realizarla. Después utilizamos la siguiente orden:

~~~
mysqldump <\base de datos> -u root -p > <\archivo de destino>
~~~

Por último, desbloqueamos el acceso a tablas.

![Creando la copia](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica5/preparacopia.png)

Posteriormente usamos SCP para copiar la base de datos, creamos la base de datos en M2 y volcamos la copia de seguridad

![Enviando la copia](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica5/copiadatabase.png)

Y por último comprobamos que todo ha ido bien.

![Enviando la copia](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica5/compruebacopia.png)

### 2.1. Replicado automático

El problema del método anterior es que en cada sincronización se necesita la acción de un administrador. Esto quizá podría automatizarse con cron, pero MySQL ya proporciona un demonio que permite la sincronización de las bases de datos de ambas máquinas **en tiempo real**.

De este modo podemos obtener una arquitectura *maestro-esclavo* en la que cuando en una de las máquinas se actualiza la base de datos, automáticamente se actualizará en la otra, o una arquitectura *maestro-maestro* en la cual esta relación es bidireccional, es decir, independientemente de en qué máquina se realicen los cambios, las dos máquinas estarán actualizadas en todo momento.

Para ello debemos configurar el servicio de MySQL de la siguiente forma:

- Hacemos que el servidor pueda escuchar a otras máquinas que no sean *localhost*.

- Definimos el archivo de log de errores y del sistema.

- Definimos el identificador del servidor

![Configurando el servidor SQL 1](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica5/confmastersql1.png)

![Configurando el servidor SQL 2](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica5/confmastersql2.png)

Posteriormente hacemos lo mismo en la máquina esclava, y una vez hecho esto, comenzamos a activar el demonio que nos permitirá construir la arquitectura maestro-esclavo.

Para ello creamos un usuario que permita que el esclavo pueda acceder a los datos del maestro, darle permisos, bloquear la base de datos, revisar los datos que identifican al maestro; y en el esclavo introducimos los datos para comunicarnos con el maestro, y establecemos la conexión.

![Configuración maestro](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica5/comandosmastersql.png)

![Configuración esclavo](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica5/comandosslavesql.png)

Comprobamos que todo está en orden:

![Arquitectura correcta](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica5/slavem2ok.png)

![Prueba maestro-esclavo](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica5/pruebamasterslave.png)

Ahora, si queremos una arquitectura maestro-maestro debemos repetir los pasos, pero invirtiendo los papeles de ambas máquinas. Así ambas máquinas son maestras y esclavas a la vez.

![Arquitectura correcta](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica5/slavem1ok.png)

![Prueba maestro-esclavo](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/practica5/pruebamastermaster.png)

Para realizar esta práctica, por simplicidad, se ha inhabilitado el firewall. Si deseásemos utilizarlo, deberíamos añadir reglas para desbloquear el puerto 3306 tanto para entrada como para salida de datos.

***

Autor: Adrián Izquierdo Pozo

Si desea ver el archivo Markdown puede verlo [en mi repositorio de Github](https://github.com/adizqpoz/SWAP/blob/master/SWAP/practica5/practica5.md)
