# Desafío de Docker compose

Ejecutar varios contenedores detras de un proxy reverso

Ver el repo [Desafio-docker-compose](https://github.com/joeldevel/sro-c2-docker-compose).

## Pasos

###  Crear entradas en `/etc/hosts` para

- wp.local
- joomla.local
- docs.local


### Crear un archivo docker-compose.yml con las siguientes secciones

#### servicio wordpress
Usamos una imagen wordpress

```yaml
    wordpress:
        image: wordpress
        restart: always
        environment:
            - WORDPRESS_DB_HOST=wordpressdb
            - WORDPRESS_DB_PASSWORD=secret
            - WORDPRESS_DB_NAME=wordpress
            - WORDPRESS_DB_USER=manager
            - VIRTUAL_HOST=wp.local
        depends_on:
            - wordpressdb
        volumes:
            - wordpress:/var/www/html
        networks: 
            - wp-net
```

#### servicio wordpressdb

```yaml

    wordpressdb:
        image: mariadb
        restart: always
        environment:
            - MYSQL_ROOT_PASSWORD=secret
            - MYSQL_DATABASE=wordpress
            - MYSQL_USER=manager
            - MYSQL_PASSWORD=secret
        volumes:
            - wordpressdb:/var/lib/mysql
        networks: 
            - wp-net
```

#### servicio joomla
Usamos una imagen bitnami/joomla.


```yaml
    joomla:
        image: bitnami/joomla
        restart: always
        environment:
            - JOOMLA_DATABASE_PORT=3306
            - JOOMLA_DATABASE_HOST=joomladb
            - JOOMLA_DATABASE_USER=bn_joomla
            - JOOMLA_DATABASE_NAME=bitnami_joomla
            - ALLOW_EMPTY_PASSWORD=yes
            - VIRTUAL_HOST=joomla.local 
            - VIRTUAL_PORT=8080
        depends_on:
            - joomladb
        ports:
            - 8080:8080
        networks:
            - joomla-net
        volumes:
            - joomla_data:/bitnami/joomla
```

#### servicio jommladb
Puede ser mysql o mariadb

```yaml
    joomladb:
        image: bitnami/mariadb
        restart: always
        environment:
            - ALLOW_EMPTY_PASSWORD=yes
            - MARIADB_DATABASE=bitnami_joomla
            - MARIADB_USER=bn_joomla
        networks:
            - joomla-net
        volumes:
            - joomladb_data:/bitnami/mariadb
```

#### Servicio para la documentación

```yaml
    docs:
        image: nginx
        environment:
            - VIRTUAL_HOST=docs.local
        volumes:
            - ./site:/usr/share/nginx/html:ro
        networks:
            - docs-net
```

#### proxy reverso

El proxy nos sirve para tener un único punto de entrada a los contenedores.

```yaml
    ingress:
        image: nginxproxy/nginx-proxy
        ports:
            - 80:80
        volumes:
            - /var/run/docker.sock:/tmp/docker.sock:ro
        networks:
            - wp-net
            - joomla-net
            - docs-net
```


#### networks

Crear networks para cada servicio


```yaml
networks:
    wp-net:
    joomla-net:
    docs-net:
```

#### volúmenes

Necesarios para persistir los datos

```yaml
volumes:
    wordpress:
    wordpressdb:
    joomla_data:
    joomladb_data:

```


##### docker-compose completo

```yaml
version: '3.8'
services:
    wordpress:
        image: wordpress
        restart: always
        environment:
            - WORDPRESS_DB_HOST=wordpressdb
            - WORDPRESS_DB_PASSWORD=secret
            - WORDPRESS_DB_NAME=wordpress
            - WORDPRESS_DB_USER=manager
            - VIRTUAL_HOST=wp.local
        depends_on:
            - wordpressdb
        volumes:
            - wordpress:/var/www/html
        networks: 
            - wp-net

    wordpressdb:
        image: mariadb
        restart: always
        environment:
            - MYSQL_ROOT_PASSWORD=secret
            - MYSQL_DATABASE=wordpress
            - MYSQL_USER=manager
            - MYSQL_PASSWORD=secret
        volumes:
            - wordpressdb:/var/lib/mysql
        networks: 
            - wp-net

    joomla:
        image: bitnami/joomla
        restart: always
        environment:
            - JOOMLA_DATABASE_PORT=3306
            - JOOMLA_DATABASE_HOST=joomladb
            - JOOMLA_DATABASE_USER=bn_joomla
            - JOOMLA_DATABASE_NAME=bitnami_joomla
            - ALLOW_EMPTY_PASSWORD=yes
            - VIRTUAL_HOST=joomla.local 
            - VIRTUAL_PORT=8080
        depends_on:
            - joomladb
        ports:
            - 8080:8080
        networks:
            - joomla-net
        volumes:
            - joomla_data:/bitnami/joomla

    joomladb:
        image: bitnami/mariadb
        restart: always
        environment:
            - ALLOW_EMPTY_PASSWORD=yes
            - MARIADB_DATABASE=bitnami_joomla
            - MARIADB_USER=bn_joomla
        networks:
            - joomla-net
        volumes:
            - joomladb_data:/bitnami/mariadb

    docs:
        image: nginx
        environment:
            - VIRTUAL_HOST=docs.local
        volumes:
            - ./site:/usr/share/nginx/html:ro
        networks:
            - docs-net

    ingress:
        image: nginxproxy/nginx-proxy
        ports:
            - 80:80
        volumes:
            - /var/run/docker.sock:/tmp/docker.sock:ro
        networks:
            - wp-net
            - joomla-net
            - docs-net
networks:
    wp-net:
    joomla-net:
    docs-net:
volumes:
    wordpress:
    wordpressdb:
    joomla_data:
    joomladb_data:

```

### Levantar los contenedores
Correr `docker compose up` para iniciar los servicios


### Verificar que funcione 
Dirigirse a las respectivas url

- [wordpress](http://wp.local)
- [joomla](http://joomla.local)
- [documentación](http://docs.local)


### Detener los contenedores 
Correr `docker compose down` para detener los servicios y remover los contenedores.

## Documentación del proyecto

Para la documentación creamos una imagen de mkdocs y le instalamos el tema material.

- crear la imagen
- crear el sitio 
- modificar el index.md con la info
- en el `mkdocs.yml` poner
  ```yaml
    site_name: Desafio Docker compose
    theme:
        name: material
  ```
- correr el build  // sitio listo para subir a cualquier servidor
- copiar el directorio `site` al directorio donde ejecutaremos `docker compose up`

## Links útiles

- [wordpress](https://hub.docker.com/_/wordpress)
- [joomla](https://hub.docker.com/r/bitnami/joomla)
- [nginxproxy](https://hub.docker.com/r/nginxproxy/nginx-proxy)
- [mkdocs](https://www.mkdocs.org/)
- [como usar nuestra mkdocs cli](https://github.com/joeldevel/sro-c1-mkdocs-cli)
