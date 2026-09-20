PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

-Entramos al puerto 80 y no parece haber nada interesante

-hacemos fuzzing y encontramos un directorio con un panel de login --> /login_page

-Probamos una Sqli (Sql Injection) sencilla haber que pasa:
	-usuario: 'or true-- -
	-Contraseña: a 
	-Conseguimos saltarnos el panel de login

-Probamos a hacer una Sqli (time based):
	-Usuario: ' or sleep(5)
	-Contraseña: Loquesea
	-FUNCIONA!! , ha tardado 5 segundos en cargar la página

-Vamos a montarnos un script en Python para explotar la Sqli (time based) y poder dumpear la información de la BBDD.

-Funcionamiento Básico:
	-Una Sqli (time based) ocurre cuando logramos inyectar el siguiente código:
		'or sleep(5) --> Lá página se queda cargando durante 5 segundos.
	-Podemos aprovecharnos de esto para dumpear toda la información de la BD, haciendo solicitudes 'POST' con letras de la 'a' a la 'z' y si tardan más de 5 segundos es que la condición se ha cumplido y la letra es correcta.

-Es posible que el servidor tarde más de 5 segundos en responder debido a la latencia. el operador 'or' tarda más tiempo que 'and'. Si sabemos el nombre de un usuario lo introducimos junto con el operador 'and' y tardará 5 segundos exactos, si utilizamos 'or' puede tardar 15 segundos.

## Ubicación Archivos SQLI (Time Based)

-Todo el paso a paso para sacar la información de la BD está en la siguiente ubicación:
	D:/Exploits_kali/Sqli_time_based








