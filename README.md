# Tarea 06

## Configuración del archivo ``.yml`` y ``.env``

`````yaml
services:
  prestashop:
    container_name: prestashop
    image: prestashop/prestashop:latest
    restart: unless-stopped
    depends_on:
       - db
    ports:
      - "${PRESTASHOP_PORT}:80"
    environment:
      DB_SERVER: ${DB_SERVER}
      DB_NAME: ${DB_NAME}
      DB_USER: ${DB_USER}
      DB_PASSWD: ${DB_PASSWD}
    volumes: # Para la persistencia de datos
      - psdata:/var/www/html
    networks:
      - prestashop_network

  db:
    container_name: mysql
    image: mysql:5.7
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_ PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - dbdata:/var/lib/mysql
    networks:
    - prestashop_network

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin
    restart: unless-stopped
    depends_on:
      - db
    environment:
      PMA_HOST: ${PMA_HOST}
      PMA_PORT: ${PMA_PORT}
      PMA_ARBITRARY: ${PMA_ARBITRARY}
    ports:
      - ${PHPMYADMIN_PORT}:80
    networks:
      - prestashop_network

volumes: # Para la persistencia de datos
  dbdata:
  psdata:

networks:
  prestashop_network:
`````
---
Documento ``.env`` para que docker-compose.yml no contenga algún dado sensible.
`````shell
# CONFIGURACIÓN BASE DATOS
MYSQL_ROOT_PASSWORD=admin
MYSQL_DATABASE=prestashop
MYSQL_USER=root
MYSQL_PASSWORD=admin
MYSQL_USER=user
MYSQL_PASSWORD=password

# CONFIGURACIÓN PRESTASHOP
DB_SERVER=db
DB_NAME=prestashop
DB_USER=root
DB_PASSWD=admin

# CONFIGURACIÓN PHPMYADMIN
PMA_HOST=db
PMA_PORT=3306
PMA_ARBITRARY=1

# PUERTOS
PRESTASHOP_PORT=8080
PHPMYADMIN_PORT=8081
`````

## Healthcheck
````yaml
    healthcheck:
      test: ["CMD","mysqladmin","ping","-h","localhost"] # Comando que se ejecuta
      interval: 10s # Tiempo de espera entre comprobaciones
      timeout: 5s # Tiempo de espera de docker por la respuesta del comando
      retries: 5 # Número máximo de intentos
````