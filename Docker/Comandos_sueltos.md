## Comandos sueltos:

        - docker images: ver las imagenes que tenemos creadas.
        
        - docker ps: ver las imagenes que tenemos activas.
                . -a: Lista todas las imagenes  
                . -q: Saca solo su ID
                
        - docker stop <nombre>: parar un contenedor.
        
        - docker rm $(docker ps -a -q): Quitar todos los procesos activos.
             
        - docker images --filter="dangling=true" -q: Ver imagenes que estan obsoletas.
        
        - docker rmi $(docker images --filter="dangling=true" -q): Borrar imágenes.
