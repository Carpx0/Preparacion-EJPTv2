Es una máquina muy sencilla que nos introduce a la vulnerabilidad IDOR.

IDOR: Ocurre cuando un sistema permite a los usuarios acceder a recursos o datos directamente a través de identificadores (como números de identificación, nombres de archivos, etc.) sin verificar adecuadamente los permisos de acceso.

-La máquina tiene abierto únicamente el puerto 80, donde aparece una imagen de 13 puertas. En el código fuente aparecen 13 hashes md5 correspondientes a cada una de las puertas. Pulsando en una de las puertas nos lleva a la ruta IP/md5-hash. 

-El reto consiste en acceder a otra puerta diferente a las que ya tenemos. Para ello hasheamos el número 0 en md5 y obtenemos la flag:

IP/cfcd208495d565ef66e7dff9f98764da

# flag{2477ef02448ad9156661ac40a6b8862e}