PORT     STATE SERVICE
80/tcp   open  http
3389/tcp open  ms-wbt-server

-Hacemos fuzzing de directorios web:
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 10.10.67.183 -t 50
	/retro

-En la ruta /retro encontramos un wordpress. 
-Utilizamos wpscan y encontramos al usuario Wade
-Para encontrar la contraseña tenemos que irnos al final de la página, dónde pone Ready Player One. Buscamos en google el nombre del avatar de Wade en la película.
	parzival

-Nos conectamos al sercicio Microsoft Remote Desktop (MSRDP), con la herramienta xfreerdp:
	xfreerdp /u:Wade /p:parzival /v:10.10.67.183



