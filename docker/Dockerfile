# ~/proint/docker/Dockerfile
# Imagen base: Ubuntu 24.04 LTS
FROM ubuntu:24.04

# Versión de PHP que corrige las vulnerabilidades críticas (security advisory de php.net)
ARG PHP_VERSION=8.5.12

# Evita preguntas interactivas de apt (solo durante el build)
ARG DEBIAN_FRONTEND=noninteractive

# Zona horaria
ENV TZ=Europe/Madrid

# 1. Herramientas base + tzdata + PPA de Ondřej Surý (versiones recientes de PHP)
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        tzdata \
        software-properties-common \
        gnupg2 \
        ca-certificates \
        lsb-release && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
    add-apt-repository -y ppa:ondrej/php && \
    apt-get update

# 2. Apache + PHP fijado a la versión corregida + extensiones para Laravel
RUN apt-get install -y --no-install-recommends \
        apache2 \
        "php8.5=${PHP_VERSION}*" \
        "libapache2-mod-php8.5=${PHP_VERSION}*" \
        "php8.5-cli=${PHP_VERSION}*" \
        "php8.5-common=${PHP_VERSION}*" \
        "php8.5-mysql=${PHP_VERSION}*" \
        "php8.5-pgsql=${PHP_VERSION}*" \
        "php8.5-sqlite3=${PHP_VERSION}*" \
        "php8.5-gd=${PHP_VERSION}*" \
        "php8.5-zip=${PHP_VERSION}*" \
        "php8.5-bcmath=${PHP_VERSION}*" \
        "php8.5-mbstring=${PHP_VERSION}*" \
        "php8.5-xml=${PHP_VERSION}*" \
        "php8.5-curl=${PHP_VERSION}*" \
        "php8.5-intl=${PHP_VERSION}*" \
        unzip \
        zip \
        git \
        curl \
    && rm -rf /var/lib/apt/lists/*

# 3. Configuración de Apache para Laravel
#    - Raíz web en la carpeta public del proyecto
#    - AllowOverride All solo en /var/www/ (necesario para el .htaccess de Laravel)
#    - ServerName para evitar el warning al arrancar
#    - mod_rewrite para las rutas de Laravel
RUN sed -i 's|/var/www/html|/var/www/html/public|g' /etc/apache2/sites-available/000-default.conf && \
    sed -i '/<Directory \/var\/www\/>/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf && \
    echo "ServerName localhost" > /etc/apache2/conf-available/servername.conf && \
    a2enconf servername && \
    a2enmod rewrite

# 4. Composer (útil para ejecutar composer/artisan dentro del contenedor)
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Directorio de trabajo (el código llega por bind mount desde compose.yaml)
WORKDIR /var/www/html

# Directorio de trabajo (el código llega por bind mount desde compose.yaml)
WORKDIR /var/www/html

# 5. Configuración de SSL y VirtualHost para diviflex
RUN a2enmod ssl headers

COPY ./docker/sites-available/diviflex.conf /etc/apache2/sites-available/diviflex.conf

RUN a2dissite 000-default && a2ensite diviflex

# Puertos expuestos para tráfico HTTP (80) y HTTPS (443)
EXPOSE 80 443

# Arranque de Apache en primer plano
CMD ["apachectl", "-D", "FOREGROUND"]
