# ProyectoApache de Diego

## Imagen necesaria

Para crear el contenedor de  **Apache** para **PHP** debemos ir a la documentación de docker donde podemos adquirir el contenedor de [Servidor Apache + PHP](https://hub.docker.com/_/php/).


## Configurando nuestro docker-compose

Antes de introducir el comando para levantar el fichero ***docker-compose.yml*** debemos introducir los siguientes datos en el fichero:

### docker-compose.yml

version: '3.9'<br>
services:<br>
  asir-apache:<br>
    image: php:7.4-apache<br>
    container_name: asir-apache<br>
    ports:<br>
    - '80:80'<br>
    - '8000:8000'<br>
    volumes:<br>
    - ./html:/var/www/html<br>
    - ./confApache:/etc/apache2<br>




* **Version**

Indicamos la version del mismo:

```
version: '3.9'
```

* **Services**

Nombre del servicio

```
services:
asir_apache:
```

* **Image & Container_name**

Nombre de nuestro contenedor y el nombre de su imagen

```
image: php:7.4-apache
container_name: asir-apache
```

* **Ports**
Los puertos donde se oirán las conexiones. Utilizaremos el puerto :80(por defecto) y el :8000
```
ports:
- '80:80'
- '8000:8000'
```
## Mis sitios

Crearemos dos directorios en nuestra carpeta raíz, sitio1 y sitio2:

sitio1>index.html_

sitio2>index.php_

## Mapeo de puertos

Para poder mapear nuestros sitios correspondientes tenemos que indicar esto en el **docker-compose.yml//volumes:**

* **Volumes**

Indicaremos que nuestro directorio "sitio1" se corresponde con la ruta /var/www/html de Apache correspondiente en sites-availables como en sites-enables. 

``` 
volumes:
- ./html:/var/www/html
- ./confApache:/etc/apache2
```
## Conexión mediante localhost

La forma que vamos a utilizar para conectarnos al arhivo html y php va a ser mediante uso de _localhost:80_    y   _Localhost:8000_

NOTA: Al introducir localhost:80 automaticamente nos lleva a localhost, pues este escucha por defecto en ese puerto.

* **Configuración de los puertos**

Como comenté anteriormente, debemos modificar el archivo **000-default.conf** y en  **DocumentRoot** añadir el nombre de nuestro directorio del archivo que queremos que se muestre al realizar la conexón _localhost_ con un puerto:

```
<VirtualHost *:80>

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html/site1
```
Con esta configuración, si nos dirigimos a nuestro navegador y escribimos _localhost:80_, encontraremos el mensaje escrito en _index.html_  **"Sitio de Diego 1"**.

Ahora, si queremos habilitar otro sitio pero con PHP, _index.php_ debemos copiar el contenido de **000-default.conf** pero cambiando el directorio root y el puerto y dejarlo en la misma carpeta con un nombre diferente, en mi caso se llama **002-default.conf** tal como a continuación:

```
<VirtualHost *:8000>
	
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html/sitio2
```

------------------------------------------------------------------------------------

## Proyecto de Apache-Virtual-Host



Vamos a  crear un dns para la pagina "maravillosas.fabulas.com" que vuelva a dirigir a nuestro cliente a un host creado por nosotros mismos.

Procedemos con la configuracion:

Utilizaremos nuestro contenedor de  apache que crearemos llamado "proyectoApache".

Lo que haremos será añadir a  nuestro docker-compose.yml el contenedor cliente alpine, el cual usaremos para conectarnos a la web, y un contenedor con bind9 para la resolución de nombres.
```
  asir_cliente:
      container_name: asir_cliente2
      image: alpine
      networks:
        bind9_subnet:
          ipv4_address: 10.1.0.224
      stdin_open: true #docker run -i
      tty: true        #docker run -t
      dns:
          - 10.1.0.222

  bind9:
    container_name: asir_bind
    image: internetsystemsconsortium/bind9:9.16
    networks:
      bind9_subnet:
        ipv4_address: 10.1.0.222
    volumes:
       - ~/Escritorio/proyectoApache-master/config:/etc/bind
       - ~/Escritorio/proyectoApache-master/zonas:/var/lib/bind
networks:
    bind9_subnet:
        external: true
 ```
Creamos la red "network" bind9 con el comando **__docker network create --subnet=10.1.0.222/24 --gateway 10.1.0.1**

Tambien añadiremos dos carpetas una config, para confirar las zonas y una zonas que gestonara la resolución de nombres

- Config
```
zone "fabulas.com" {
        type master;
        file "/var/lib/bind/db.fabulas.com";
        allow-query{
            any;
            };
};
```
```
options {
        directory "/var/cache/bind";
        forwarders {
                8.8.8.8;
                8.8.4.8;
        };
        forward only;

        listen-on { any; };
        listen-on-v6 { any; };
        allow-query {
                any;
        };
        allow-recursion {
                none;
        };
        allow-transfer {
                none;
        };
        allow-update {
                none;
        };
};
```  
- Zonas
```

$TTL    3600
@       IN      SOA     ns.fabulas.com. root.fabulas.com. (
                   2007010401           ; Serial
                         3600           ; Refresh [1h]
                          600           ; Retry   [10m]
                        86400           ; Expire  [1d]
                          600 )         ; Negative Cache TTL [1h]
;
@       IN      NS      ns.fabulas.com.
ns       IN      A       10.1.0.222

maravillosas     IN      A       10.1.0.225
oscuras     IN      CNAME       maravillosas
```
A continuacion nos dirigimos a nuestra carpeta Sites-enable y añadimos nuestro server name como a continuación. Para conectarlo con el puerto indicado en mi caso el 80:



Si hemos hecho todo bien ya podremos hacer un docker-compose up y ya lo tendremos listo.



Ahora para comproba que funciona debemos entrar en la shell de nuestro cliente e intentaremos hacer ping a la ip nuestro apache y a la ip nuestro de servidor dns, el cual nos debería dar respuesta si tenemos todo bien.



Una vez comprobamos que tenemos conexión con el dns ya podremos hacer peticiones a la página y recebiremos la información que introduimos dentro imagen

## Configurar SSL

Para empezar nos dirigimos a  attach shell del apache y escribimos el comando **__"a2enmod ssl"__**, esto activara la configuración de nustro certificados en nuestro contenedor.

A continuación creamos la carpeta **__"certs"__**, donde guardaremos nuestros certificados, y nos movemos a dicha carpeta  

Ahora para completar dicha configuración  utilizamos el comando **__"openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out apache-certificate.crt -keyout apache.key"__** y rellenamos los campos como se indique

Ahora debemos avtivar la configuración Modificar el fichero default-ssl.conf con:
a2ensite default-ssl.conf

Tambien creamos una carpeta SSL, con un index dentro que sera lo que aparecera cuando cargemos el host

- **Modificacionn del fichero default-ssl.conf**

```

    DocumentRoot /var/www/html/ssl --- Nueva ruta
```
    


    SSLCertificateFile    /etc/apache2/certs/apache-certificate.crt  --- Nuevas rutas
    SSLCertificateKeyFile /etc/apache2/certs/apache.key


-  *Recordar Editar docker para añadir puerto 443*
- Si hemos completado todos los pasos correctamente nuestro ssl ya deber


## Añadir firefox

Para añadir firefox lo que vamos a hacer es añadir las siguientes lineas de codigo
```
  firefox:
    container_name: firefox
    image: jlesage/firefox
    ports:
      - '5800:5800'
    volumes:
      - ./Firefox:/config:rw
    dns:
      - 10.1.0.222
    networks:
      bind9_subnet:
        ipv4_address: 10.1.0.233
 ```
 Tambien creamos una nueva carpeta firefox donde se van a guardar los ficheros del volumen
 **__mkdir firefox__**
 


## Añadir Wireshark

Para añadir un contenedor de Wireshark añadimos el siguiente codigo a nuestro codigo a nuestro **__Docker Compose__**

```
  wireshark:
    image: lscr.io/linuxserver/wireshark:latest
    container_name: wire
    cap_add:
      - NET_ADMIN
    security_opt:
      - seccomp:unconfined 
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - ./wires:/config
    ports:
      - 3000:3000
    restart: unless-stopped
```

 Tambien creamos una nueva carpeta firefox donde se van a guardar los ficheros del volumen
 **__mkdir wires__**





Diego Varela Vidal
