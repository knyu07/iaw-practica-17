# Práctica 17: Balanceo de carga con HAProxy

En esta práctica vamos a incluir un nuevo contenedor Docker con HAProxy para balancear la carga de los contenedores que ejecutan la aplicación web. 

## Docker Compose

Para el archivo docker-compose.yml necesitaremos: 

- Balanceo de carga
- MySQL
- phpmyadmin
- Apache

### Balanceo de carga (HAProxy)

Utilizaremos la imagen dockercloud/haproxy de DockerHub, añadiremos los puertos 80 y 1936 (abrir puertos en AWS) que nos permite acceder a una página con información del balanceo.
Creamos un enlace con el servicio que queremos balancear. Los enlaces permiten que los contenedores se descubran entre sí y transfieran de manera segura información sobre un contenedor a otro contenedor y para finalizar montaremos el socket UNIX del Docker daemon (/var/run/docker.sock) para que el contenedor lb pueda comunicarse con el Docker daemon y obtener información del resto de contenedores.

```
lb:
    image: dockercloud/haproxy 
    ports:
        - 80:80 
        - 1936:1936 
    links:
        - apache 
    volumes:
        - /var/run/docker.sock:/var/run/docker.sock 
    networks:
        - frontend-network
```

### Apache

Cogeremos de la práctica 16 la parte de apache, aunque haremos una modificación. Dentro de apache tendremos que quitar el puerto señalado (80:80). Como se puede ver ya está señalado dentro del balanceador, ya que si queremos iniciar varios apaches no nos dejaría porque todos estarían escuchando al mismo puerto.

```
 apache: 
        build: ./apache
        depends_on: 
            - mysql
        networks: 
            - frontend-network
            - backend-network
        restart: always
```

Para phpmyadmin y MySQL cogeremos la de la práctica anterior (no hay ningún cambio). Seguidamente creamos nuestro Dockerfile de nuestra imagen LAMP, nuestro archivo oculto .env para las variables del docker-compose y nuestra database.sql. 

Para iniciar varios apaches hacemos uso del comando: **docker-compose up --scale apache=4** 
