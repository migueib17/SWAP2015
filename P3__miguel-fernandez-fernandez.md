

1. Instalación y configuración de nginx.

     Lo primero que tenemos que hacer es importar la clave:

        wget http://nginx.org/keys/nginx_signing.key
        apt-key add /tmp/nginx_signing.key
        rm -f /tmp/nginx_signing.key

*Queda añadido el repositorio al fichero /etc/apt/sources.list 


2. Instalamos el paquete nginx.

    apt-get install nginx


3. Vamos a configurar nginx como balanceador de carga, definimos que maquinas queremos añadir. Primero vamos a editar el archivo de configuración:

    sudo nano /etc/nginx/conf.d/default.conf

y el archivo se queda así:
------------------------------------------------------------------------------------------

upstream apaches {
    server 192.168.1.34;
    server 192.168.1.35;

}

server{
    listen 80;
    server_name balanc;

    access_log /var/log/nginx/balanc.access.log;
    error_log /var/log/nginx/balanc.error.log;
    root /var/www/;

    location / {
        proxy_pass http://apaches;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Connection ÒÓ;
    }

}
------------------------------------------------------------------------------------------


4. Comprobamos el funcionamiento:

    sudo service nginx restart
    curl http://192.168.1.36



5. Instalaci—n y configuraci—n de haproxy. Instalamos el paquete con la siguiente orden:

    apt-get install haproxy



6. Editamos el fichero /etc/haproxy/haproxy.cfg y aplicamos nuestra configuraci—n:

---------------------------------------------------------
global
        daemon
        maxconn 256

defaults
        mode http
        contimeout 4000
        clitimeout 42000
        srvtimeout 43000

frontend http-in
        bind *:80
        default_backend servers

backend servers
        server 		server1 192.168.1.34:80 maxconn 32
        server 		server2 192.168.1.35:80 maxconn 32
---------------------------------------------------------



7. Lanzamos el servicio:

    sudo /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg



8. Comprobamos el funcionamiento mediante la orden curl o tambiŽn podemos hacerlo desde el navegador de la m‡quina f’sica.

    curl http://192.168.1.36

