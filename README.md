# 🐳 Tarea06 - Instalación de PrestaShop con Docker Compose

> [!NOTE]
> Imágenes utilizadas: [PrestaShop](https://hub.docker.com/r/prestashop/prestashop/), [MySQL](https://hub.docker.com/_/mysql) y [phpMyAdmin](https://hub.docker.com/_/phpmyadmin)


Archivo `docker-compose.yml` con healthcheck en la base de datos, haciendo que los demás servicios esperen a que esté completamente operativa. Utiliza un archivo `.env` para guardar la información sensible, volúmenes para mantener la persistencia de datos y una instalación automática de PrestaShop, sin necesidad de pasar por el asistente de instalación.
`````yaml
services:
  prestashop:
    container_name: prestashop
    image: prestashop/prestashop:latest
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy       # Espera a que la base de datos responda
    environment:
      DB_SERVER: ${DB_SERVER}
      DB_NAME: ${DB_NAME}
      DB_USER: ${DB_USER}
      DB_PASSWD: ${DB_PASSWD}
      PS_INSTALL_AUTO: 1                 # Instala PrestaShop automáticamente
      PS_DOMAIN: localhost:8080
      PS_LANGUAGE: es
      PS_COUNTRY: ES
      ADMIN_MAIL: ${ADMIN_MAIL}
      ADMIN_PASSWD: ${ADMIN_PASSWD}
    ports:
      - ${PRESTASHOP_PORT}:80
    volumes:
      - psdata:/var/www/html                                 # Volumen para la persistencia de datos
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
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - prestashop_network
    healthcheck:                                              # Comprueba que la base de datos está operativa
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]  # Comando que comprueba la conexión
      interval: 10s                                           # Intervalo entre comprobaciones
      timeout: 5s                                             # Tiempo máximo de espera por respuesta 
      retries: 5                                              # Número máximo de intentos
      start_period: 20s                                       # Tiempo de espera inicial antes de iniciar la verificación

  phpmyadmin:
    container_name: phpmyadmin
    image: phpmyadmin/phpmyadmin
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy  # Espera también al healthcheck de la base de datos
    environment:
      PMA_HOST: ${PMA_HOST}
      PMA_PORT: ${PMA_PORT}
      PMA_ARBITRARY: ${PMA_ARBITRARY}
    ports:
      - ${PHPMYADMIN_PORT}:80
    networks:
      - prestashop_network

volumes:                                                      # Volúmenes para mantener los datos 
  dbdata:
  psdata:

networks:                                                      # Red interna compartida entre los servicios
  prestashop_network:

`````
---
Archivo `.env` para que `docker-compose.yml` no contenga información sensible ni contraseñas directamente.
`````shell
# CONFIGURACIÓN BASE DATOS
MYSQL_ROOT_PASSWORD=admin
MYSQL_DATABASE=prestashop
MYSQL_USER=psuser
MYSQL_PASSWORD=pspass

# CONFIGURACIÓN PRESTASHOP
DB_SERVER=db
DB_NAME=prestashop
DB_USER=psuser
DB_PASSWD=pspass

# CONFIGURACIÓN PHPMYADMIN
PMA_HOST=db
PMA_PORT=3306
PMA_ARBITRARY=1

# PUERTOS
PRESTASHOP_PORT=8080
PHPMYADMIN_PORT=8081
`````

----
### Comandos utilizados:
- `docker compose up` Inicia los contenedores (sin `-d` para ver los logs).
- `docker compose down -v` Detiene y elimina los contenedores y volúmenes.

---

### Captura final
Se muestra **PrestaShop** funcionando directamente sin pasar por el asistente de instalación y **phpMyAdmin** mostrando las tablas
de la base de datos prestashop.

![1.PNG](img%2F1.PNG)
