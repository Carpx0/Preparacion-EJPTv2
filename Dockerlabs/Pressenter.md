PORT   STATE SERVICE
80/tcp open  http

-Entramos a la web y no vemos nada interesante, hacemos fuzzing y no hay nada

-Nos fijamos que abajo del todo pone un dominio 'pressenter.hl', parece que se está aplicando virtual hosting y tienen otra web

-Lo añadimos al /etc/hosts y ya nos deja entrar, es un Wordpress

-Enumeramos con Wpscan y encontramos los usuarios hacker y pressi

-Aplicamos fuerza bruta con Wpscan al usuario pressi y encontramos su contraseña

-Estamos dentro del wordpress, no podemos editar los templates, asi que subimos un plugin .zip con la reverse shell dentro para ganar acceso:
	/*
	        Plugin Name:  Pwned                     
	        Plugin URI:   https://www.wpbeginner.com
	        Description:  A short little description of the plugin. It will be displayed on the Plugins page in WordPress admin area.
	        Version:      1.0
	        Author:       WPBeginner
	        Author URI:   https://www.wpbeginner.com  
	a*/ 
	-php exec ... 

www-data@f30856b72142:

## Escalada de Privilegios

-No podemos ejecutar comando como sudo ni tenemos binarios SUID interesantes

-Nos vamos a /var/www/pressenter/wp-config.php y vemos las credenciales para entrar a mysql:
	/** Database username */
	define( 'DB_USER', 'admin' );
	/** Database password */
	define( 'DB_PASSWORD', 'rooteable' );

-Entramos a mysql en local y vemos la contraseña del usuario  'enter':
	-mysql -uadmin -prooteable
	-mysql> select * from wp_usernames;
+----+----------+-----------------+---------------------+
| id | username | password        | created_at          |
+----+----------+-----------------+---------------------+
|  1 | enter    | kernellinuxhack | 2024-08-22 13:18:04 |
+----+----------+-----------------+---------------------+

-Pivotamos al usuario 'enter':
	-su enter
	-password: kernellinuxhack
	enter@f30856b72142:/$

-enter@f30856b72142:/$ sudo -l
	(ALL : ALL) NOPASSWD: /usr/bin/cat
    (ALL : ALL) NOPASSWD: /usr/bin/whoami

-Intentamos buscar un archivo que contenga la contraseña de root pero no encontramos nada

-Alfinal la contraseña de root era la misma que la de enter:
	-su root
	-password: kernellinuxhack

