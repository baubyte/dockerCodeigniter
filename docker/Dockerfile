
FROM php:8.1.12-apache

LABEL name="PHP 8.1 última versión estable en español + Apache"

#Varibles de Entorno
ENV APP_ENV=development \
    APP_BASE_DIR=/var/www/ \
    APP_SERVER_ADMIN=admin@baubyte.ar \
    APP_TIMEZONE=America/Argentina/Buenos_Aires \
    APACHE_RUN_USER=devbaubyte \
    APACHE_LOG_DIR=/web/apache/logs


# Instalar dependencias del sistema
RUN apt-get update && apt-get install -y \
    bash-completion \
    libpq-dev \
    less \
    locales \
    multitail \
    nano \
    ssl-cert \
    tzdata \
    wget \
    curl \
    libbz2-dev \
    libfreetype6-dev \
    libicu-dev \
    libjpeg-dev \
    libonig-dev \
    libzip-dev \
    libmcrypt-dev \
    libpng-dev \
    libreadline-dev \
    sudo \
    unzip \
    zip \
    && localedef -i es_ES -c -f UTF-8 -A /usr/share/locale/locale.alias es_ES.UTF-8

# \ && make-ssl-cert generate-default-snakeoil

#Establecemos el lenguaje 
ENV LANG es_ES.UTF-8

#Establecemos la hora local
RUN rm /etc/localtime
RUN echo $APP_TIMEZONE > /etc/timezone \
    && ln -snf /usr/share/zoneinfo/$APP_TIMEZONE /etc/localtime \
    && echo $APP_TIMEZONE > /etc/timezone

# Limpiar cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

#Permisos y hora local
RUN a2enmod rewrite \
    && true \
    && a2enmod ssl
RUN printf '[Date]\ndate.timezone="%s"\n', $APP_TIMEZONE > /usr/local/etc/php/conf.d/tzone.ini



# Instalar extensiones PHP
RUN docker-php-ext-install \
    bcmath \
    gd \
    bz2 \
    calendar \
    iconv \
    intl \
    mbstring \
    opcache \
    pdo_mysql \
    mysqli \
    zip

# Obtener la ultima version de Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer


# Agregamos Node 18 LTS
RUN curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key \
     | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg; \
    NODE_MAJOR=18; \
    echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" \
     > /etc/apt/sources.list.d/nodesource.list; \
    apt-get -qy update; \
    apt-get -qy install nodejs;

# SSL
COPY config/cert.pem /etc/ssl/certs/cert.pem
COPY config/cert-key.pem /etc/ssl/private/cert-key.pem

# virtual Host
COPY config/000-default.conf /etc/apache2/sites-enabled/000-default.conf

# Necesitamos un usuario con el mismo UID / GID que el usuario host
# Entonces, cuando ejecutamos los comandos de CLI, todos los permisos y 
# la propiedad del archivo de host permanecen intactos.
# Los comandos de lo contrario desde el interior del contenedor 
# crearían archivos y directorios de propiedad de la raíz. 
ARG uid
RUN useradd -G www-data,root -u $uid -d /home/"${APACHE_RUN_USER}" "${APACHE_RUN_USER}"
RUN mkdir -p /home/"${APACHE_RUN_USER}"/.composer && \
    chown -R "${APACHE_RUN_USER}":"${APACHE_RUN_USER}" /home/"${APACHE_RUN_USER}"

# Establecer directorio de trabajo
WORKDIR ${APP_BASE_DIR}
EXPOSE 80
EXPOSE 443
