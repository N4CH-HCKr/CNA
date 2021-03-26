## Comandos sueltos:

        - docker images: ver las imagenes que tenemos creadas.
        
        - docker cp <ruta archivo local> id:<ruta contenedor>
        
        - docker ps: ver las imagenes que tenemos activas.
                . -a: Lista todas las imagenes  
                . -q: Saca solo su ID
                
        - docker stop <nombre>: parar un contenedor.
        
        - docker rm $(docker ps -a -q): Quitar todos los procesos activos.
             
        - docker images --filter="dangling=true" -q: Ver imagenes que estan obsoletas.
        
        - docker rmi $(docker images --filter="dangling=true" -q): Borrar imágenes.

        - Obtener IP's:
                . docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq) --> Todos                             
                . docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name_or_id --> Uno en concreto
                . docker inspect CONTAINER_ID | grep "IPAddress"
        
        - Reenvío de puertos: -p<puerto_local>:<puerto_remoto>
