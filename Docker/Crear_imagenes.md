1º Instalamos docker:

        - sudo apt-get install docker.io

2º Matamos los procesos que levanta por defecto:

        - sudo pkill dockerd
        - sudo pkill docker

3º Levantamos el proceso dockerd en segundo plano:

        - dockerd > /dev/null 2>&1 &
        - disown % --> Independizar el proceso de la ventana

4º Crear una imagen:

        - Crear un archivo: Dockerfile
        - Ejemplo de creación de imagen de un ubuntu (Archivo Dockerfile)
        - Comando: sudo docker build . -t <nombre> 

5º Lanzamos la imagen:

        - sudo docker run -dit --network bridge
                . -d: Ponerlo en segundo plano.
                . -it: modo interactivo.
                . --network bridge: Adaptador de red en modo puente.
                . --name <nombre a darle> <nombre imagen>: darle un nombre al contenedor.
        - Se crea el identificador 

6º Interactuar con el contenedor

        - docker exec -it <nombre contenedor> bash
                . bash: El tipo de terminal que tiene.

## Comandos sueltos:
        - docker images: ver las imagenes que tenemos creadas.
        - docker ps: ver las imagenes que tenemos activas.
                . -a: Lista todas las imagenes  
                . -q: Saca solo su ID
        - docker stop <nombre>: parar un contenedor.
        - docker rm $(docker ps -a -q): Quitar todos los procesos activos.     
        - docker images --filter="dangling=true" -q: Ver imagenes que estan obsoletas.
        - docker rmi $(docker images --filter="dangling=true" -q): Borrar imágenes.
