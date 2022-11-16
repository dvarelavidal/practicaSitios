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




Diego Varela Vidal