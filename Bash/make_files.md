#!/bin/bash

#Script creado para crear ficheros con un Shebang personalizado y los permisos que se quieran.
#Creado by N4CH-HCKr
#Mejorando skills en bash

echo -e -n "\e[1;37m Nombre del fichero: \e[1;32m\e "
read fichero
echo -e -n "\e[1;37m Formato del fichero: \e[1;32m\e "
read formato
archivo=$fichero$formato
 
echo -e -n "\e[1;37m Quieres guardarlo en el directorio actual? (S/N) \e[1;32m\e "
read si
if [[ $si == S ]]; then
        touch $(pwd)/$archivo
        echo -e "\e[0;31m Guardado en $(pwd) \e[1;37m\e "
        echo ""
else
        echo -e -n "\e[1;37m Escribe la ruta donde quieres guardarlo: \e[0;31m\e "
        read ruta
        touch $ruta/$archivo
        echo "\e[0;31m Guardado en $ruta \e[1;37m\e "
        echo ""
fi
 
echo -e -n "¿Quieres modificar alguna característica? (S/N) \e[1;32m\e "
read si
echo ""
 
while [ $si == S ]; do
 
        echo -e "\e[1;37m Elija alguna opción:"
        echo ""
        echo -e "\t1) Añadir shebang.\n\t2) Cambiar permisos.\n\t3) Finalizar.\n \e[1;32m\e "
 
        echo -e -n ">="
        read opciones
                case $opciones in
 
                1)
                                read -p "Añada el SHEBANG: " shebang
                                echo "$shebang" >> /home/n4ch/OSCP/Bash/pruebas/$archivo
                                echo -e "\e[0;31m Shebang añadido a $archivo \e"
                                echo ""
                ;;
                2)
                                read -p "Escriba los permisos: " permisos
                                chmod $permisos /home/n4ch/OSCP/Bash/pruebas/$archivo
                                echo -e "\e[0;31m El fichero $archivo ahora tiene permisos $permisos \e"
                                echo ""
                ;;
                3)
                                echo "Generando el fichero"
                                break
                esac
done
 
echo ""
echo -e "\e[1;37m --> Fichero $archivo creado correctamente \e"
