# Wireshark

Wireshark es una herramienta que permite visualizar toda petición que se transporta por una determinada red.

Lo que se nos pide es analizar el flujo de ejecución de las peticiones HTTP que le solicitamos al balanceador para que nos responda nuestra granja web.

Para ello, vamos a arrancar nuestra M1, M2 y M3, y vamos a ejecutar wireshark:

![Inicio Wireshark](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/Ejercicios/iniciowireshark.png) 

Debemos seleccionar la red que gestiona nuestra granja web. En nuestro caso es vboxnet0.

![Viendo nuestra red](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/Ejercicios/redgranja.png) 

Como vemos, hay algún que otro mensaje irrelevante. Sin embargo, como la cantidad es despreciable, lo obviaremos. Si fuese necesario, lo que haríamos es filtrar las peticiones según el protocolo que se utilice, pero este no es el caso.

Ahora vamos a proceder a hacer tres peticiones HTTP a nuestro balanceador de carga. Veremos lo siguiente:

![Viendo nuestra red](https://raw.githubusercontent.com/adizqpoz/SWAP/master/SWAP/Ejercicios/peticiones.png) 

Como vemos, ahora hay una cantidad de peticiones significativa. Si las observamos, vemos que para cada petición existe la siguiente secuencia de peticiones:

	- Petición TCP de sincronización desde el cliente hacia M3.
	
	- Petición TCP de sincronización desde M3 hacia el cliente, con la bandera ACK levantada.
	
	- Petición TCP de confirmación de que ACK está levantada desde el cliente hacia el balaceador.
	
	- Petición HTTP desde el cliente hacia el balanceador.
	
	- Petición Broadcast ARP en el que M3 pregunta quién es M1 para redirigirle el tráfico.
	
	- Respuesta HTTP con código 200 que indica que es una petición correcta, que contiene el contenido HTML.
	
	- Respuesta TCP de parte del cliente en el cual se notifica que se ha recibido la respuesta HTTP.
	
	- Petición por parte del cliente en la cual se solicita el fin de la conexión.
	
	- Respuesta del balanceador relativa al fin de la conexión.
	
	- Notificación del cliente en el que se indica que la conexión ha sido finalizada.
	
Como podemos ver, vemos cómo se realiza la interacción entre el balanceador y el cliente. Sin embargo, no somos capaces de ver la redirección de tráfico desde M3 hacia M1 o M2. Sin embargo, sí que vemos una petición ARP en la cual el balanceador pregunta quién es M1, con lo cual podemos intuir que se redirige el tráfico a esa máquina, o como mínimo se le solicita algún dato relativo a la petición.

***

Autor: Adrián Izquierdo Pozo
