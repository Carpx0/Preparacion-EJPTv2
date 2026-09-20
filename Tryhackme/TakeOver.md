Máquina útil para aprender fuzzing de subdominios pero nada más.

PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

-Hay que añadir el siguiente dominio al /etc/hosts:
futurevera.thm

-Haciendo fuzzing de directorios no encontramos nada
-Encontramos 2 nuevos dominios haciendo fuzzing de subdominios:
	Gobuster no funciona: Debido a que el modo vhost no tiene la opción de filtrar por código de estado HTTP. Aun usando el mismo diccionario que ffuf, gobuster no lo encuentra.

Con ffuf si lo encuentra: ffuf -u https://futurevera.thm/ -w /usr/usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -mc all -t 10 -H "Host: FUZZ.futurevera.thm" -fs 4605

Explicación parametros:
-mc all: Define que todos los códigos de estado HTTP serán considerados como válidos. Gracias a este comando obtenemos los subdominios, ya que devuelven un codigo 421, el cual no sale si no especificas este parámetro.
-fs 4605: Descarta las respuestas HTTP cuyo tamaño sea 4605, esto se debe a que salían todo el rato respuestas inválidas con esa longitud de bytes. 

-Añadimos los subdominios encontrados al /etc/hosts

-Buscamos el dominio support.futurevera.thm. 

-Hacemos click en el candado de https -> connection not secure -> more information 
-> view certificate

-Copiamos el DNS name: secrethelpdesk934752.support.futurevera.thm

Hay que pegarlo como http y sale la flag:
http://secrethelpdesk934752.support.futurevera.thm