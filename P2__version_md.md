

0.Direcciones ip de ambas máquinas

	ifconfig:

	-Máquina1:	192.168.1.50
	-Máquina2:	192.168.1.49

1. Crear un tar con ficheros locales en un equipo remoto.
	
	Ejecutamos en Máquina1 para copiar en Máquina2:

	tar czf - ./home/migueib17 | ssh 192.168.1.49 'cat > ~/tar.tgz'

*Al ejecutar este comando tuve un problema con el ssh: "conection refused port 22".
Solucioné el problema fácilmente actualizando ssh en las dos máquinas: sudo apt-get install ssh y puse las dos máquinas virtuales de vmware en puente.

2. Instalar la herramienta rsync

	En ambas máquinas:
	1. sudo su
	2. contraseña
	3. apt-get install rsync 	--> (ya lo tenía actualizado)
	
	En máquina2:
	4. rsync -avz -e ssh root@192.168.1.50:/var/www/ /var/www/
	(clona la carpeta de la máquina1 en nuestra máquina).

*Al ejecutar este comando tuve un problema: "protocol version mismatch -- is your shell clean?". Este problema se debe a que tengo el prompt modificado, entonces tuve que dejar el archivo .bashrc como venía de fábrica, para que ssh no me diera problemas.

*queda clonado el directorio

3. Configuramos ssh para acceder sin contraseña

	En la máquina2:
	1. ssh-keygen -t dsa					->Generamos la clave ssh
	2. ssh-copy-id -i .ssh/idsa.pub root@192.168.1.50	->Copiamos la clave a la máquina1

*Ahora podemos acceder a la máquina1 sin contraseña.

4. Programamos las tareas con crontab

	En la máquina2:
	1. Editamos el fichero /etc/crontab
	2. Añadimos la siguiente línea: 0 *   * * *   root rsync -avz -e ssh root@192.168.1.50:/var/www/ /var/www/

*Se ejecutará la herramienta rsync para la clonación de la carpeta /var/www/ en el minuto 0 de cada hora para siempre.

