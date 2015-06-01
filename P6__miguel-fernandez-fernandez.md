#Práctica 6. Discos en RAID
##Miguel Fernández Fernández



Primero añadimos dos nuevos discos duros a la máquina virtual sobre los que haremos el Raid.

#Configuración del RAID por software

Arrancamos la máquina y entramos para instalar el software necesario para
configurar el RAID:
    
    sudo apt-get install mdadm


Debemos buscar la información (identificación asignada por Linux) de ambos discos:
    
    sudo fdisk -l


Ahora ya podemos crear el RAID 1, usando el dispositivo /dev/md0, indicando el
número de dispositivos a utilizar (2), así como su ubicación:

    sudo mdadm -C /dev/md0 --level=raid1 --raid-devices=2 /dev/sdb /dev/sdc


Una vez creado el dispositivo RAID, le damos formato:

    sudo mkfs /dev/md0


Ahora ya podemos crear el directorio en el que se montará la unidad del RAID:

    sudo mkdir /datos
    sudo mount /dev/md0 /datos


Para comprobar el estado del RAID, ejecutaremos:

    sudo mdadm --detail /dev/md0


En el fichero /etc/fstab añadimos lo siguiente para que cuando se reinicie el sistema automáticamente se monte el RAID, y con la orden mount podemos ver si se ha montado correctamente:

    .# echo "UUID=xxxx-xxxx-xxxx-xxxx /datos ext2 defaults 0 0" >> /etc/fstab


Con la siguiente orden podemos ver el estado del RAID:

    mdadm --detail /dev/md127


