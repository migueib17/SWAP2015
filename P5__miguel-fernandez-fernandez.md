#Práctica 5. Replicación de bases de datos MySQL
##Miguel Fernández Fernández


    Máquina1 ---> 192.168.1.50      BD original
    Máquina2 ---> 192.168.1.49      réplica


#Crear una BD e insertar datos

Utilizando la máquina1 nos conectamos a MySQL:

	mysql -u root -p

Creamos una BD y una tabla e insertamos un registro:
	
	create database usuarios;

	use usuarios;

	create table registrados(nombre varchar(10), pass varchar(6));

	insert into registrados(nombre, pass) values("Ernesto","123456");

    describe registrados;

    select * from registrados;

    exit


#Replicar una BD MySQL con mysqldump

Bloqueamos las tablas de la BD en la máquina1 para que en este paso no se pueden insertar más datos y para que no se pueda acceder a ellas.

    mysql -u root -p

    FLUSH TABLES WITH READ LOCK;
    
    exit;

Hacemos una copia de la BD:

	mysqldump  bd_1 -u root -p > bd_1.sql
	
Desbloquear las tablas que habíamos bloqueado anteriormente:

    mysql -u root -p

    UNLOCK TABLES;
    
    exit;


## Restaurar réplica creada con mysqldump

Ya podemos ir a la máquina esclavo (maquina2, secundaria) para copiar el archivo 
    .SQL con todos los datos salvados desde la máquina principal (maquina1):

    scp root@maquina1:/root/bd_1.sql /root/

Ahora restauramos la copia dentro de la base de datos de esta máquina. 

La orden mysqldump no incluye en ese archivo la sentencia para crear la BD (es necesario que nosotros la creemos en la máquina secundaria en un primer paso, antes de restaurar las tablas de esa BD y los datos contenidos en éstas).

Creamos la BD antes de restaurar:

    mysql -u root -p

    create database bd_1;

    exit;


Ahora si podemos restaurar los datos sin ningún problema. 

    [mysql -u root -p bd_1 < /root/bd_1.sql]


Por supuesto, también podemos hacer la orden directamente usando un “pipe” a un ssh para exportar los datos al mismo tiempo (siempre y cuando en la máquina secundaria ya hubiéramos creado la BD):

    mysqldump bd_1 -u root -p | ssh equipodestino mysql


# Replicación de BD mediante una configuración maestro-esclavo

Lo primero es añadir la configuración al fichero /etc/mysql/my.cnf. Y reiniciamos el servicio mysql.

##Configuración de mysql del maestro:

    [#bind-address 127.0.0.1]

Le indicamos donde guardar un log para los errores que surjan:

    log_error = /var/log/mysql/error.log

Establecemos el identificador del servidor:
    
    server-id = 1

Indicamos el registro con la información del registro de actualizaciones:

    log_bin = /var/log/mysql/bin.log


Guardamos el documento y reiniciamos el servicio:
    
    /etc/init.d/mysql restart

Si no nos ha dado ningún error la configuración del maestro, podemos pasar a hacer la configuración del mysql del esclavo (vamos a editar como root el /etc/mysql/my.cnf).


##Configuración de mysql del esclavo:

Comprobamos versión de mysql:

    mysql --version

La configuración es similar a la del maestro, con la diferencia de que el server-id en esta ocasión será 2.

    server-id = 2

Reiniciamos el servicio en el esclavo:

    /etc/init.d/mysql restart

Si no da ningún error podemos volver al maestro para crear un usuario y darle permisos de acceso para la replicación. 

Entramos en mysql y ejecutamos las siguientes sentencias:

    mysql> CREATE USER esclavo IDENTIFIED BY 'esclavo';
    mysql> GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%' IDENTIFIED BY 'esclavo';
    mysql> FLUSH PRIVILEGES;
    mysql> FLUSH TABLES;
    mysql> FLUSH TABLES WITH READ LOCK;


Obtenemos los datos de la base de datos que vamos a replicar para posteriormente usarlos en la configuración del esclavo:
    
    mysql> SHOW MASTER STATUS;

Estos datos se pueden introducir directamente en el archivo de configuración si trabajamos con versiones inferiores a mysql 5.5. Si no es así, en el entorno de mysql ejecutamos la siguiente sentencia:

    mysql> mysql> CHANGE MASTER TO MASTER_HOST='192.168.1.50',
    MASTER_USER='esclavo', MASTER_PASSWORD='esclavo',
    MASTER_LOG_FILE='bin.000001', MASTER_LOG_POS=501,
    MASTER_PORT=3306;

Arrancamos el esclavo:

    mysql> START SLAVE;

Volvemos al maestro y volvemos a activar las tablas para que puedan meterse nuevos datos en el maestro:

    mysql> UNLOCK TABLES;

Si queremos asegurarnos de que todo funciona perfectamente y que el esclavo no tiene ningún problema para replicar la información, nos vamos al esclavo y con la siguiente orden:

    mysql> SHOW SLAVE STATUS\G


Revisamos si el valor de la variable “Seconds_Behind_Master” es distinto de “null”. En ese caso, todo estará funcionando perfectamente.

Si no es así, es que hay algún error y los valores de las demás variables indicarán cuál es el problema y cómo arreglarlo para que todo funcione perfectamente. A continuación vemos que el valor de la variable “Seconds_Behind_Master” es 0, por lo que no hay ningún error y todo funciona como debe.

Para comprobar que todo funciona, debemos ir al maestro e introducir nuevos datos a la base de datos. A continuación vamos al esclavo para revisar si la modificación se ha reflejado en la tabla modificada en el maestro.
	



