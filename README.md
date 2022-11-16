proyectoApache de Diego Varela:


Para el contenedor de apache, lo descargamos desde la pagina de php.



Para levantar el servicio, utilizaremos el docker-compose y para detenerlo utilizamos docker-compose down




Para levantar las paginas debemos crear un nuevo directorio para cada sitio, llamados sitio1 y sitio2.


Debemos modificar en los sites-available los 000-default.conf en la parte superior el puerto por el cual escuchará nuestro sitio y la ruta en la cual se encuentra nuestra carpeta.
Haremos exactamente lo mismo con el sitio 002-default.conf, pero a este le añadiremos que escuchará por el puerto 8000.





