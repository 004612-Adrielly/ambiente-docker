# Apache atualizado + PHP 8.2.4 (mod_php)
FROM php:8.2.4-apache

# Pacotes base e libs para compilar/extensões
RUN apt-get update && apt-get install -y --no-install-recommends \
    gnupg2 apt-transport-https ca-certificates curl unzip \
    libzip-dev libicu-dev libxml2-dev \
    libpng-dev libjpeg-dev libfreetype6-dev \
    unixodbc unixodbc-dev \
    && rm -rf /var/lib/apt/lists/*

# Extensões PHP comuns (ajuste conforme seu projeto)
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
       gd intl zip bcmath mysqli pdo pdo_mysql

# Drivers Microsoft SQL Server (sqlsrv 5.12.x = PHP 8.2; latest pecl exige PHP >= 8.3)
RUN set -eux; \
    . /etc/os-release; \
    curl -fsSL https://packages.microsoft.com/keys/microsoft.asc \
      | gpg --dearmor -o /usr/share/keyrings/microsoft-prod.gpg; \
    if [ "$ID" = "debian" ] && [ "$VERSION_CODENAME" = "bookworm" ]; then \
        echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft-prod.gpg] https://packages.microsoft.com/debian/12/prod bookworm main" \
          > /etc/apt/sources.list.d/mssql-release.list; \
    else \
        echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft-prod.gpg] https://packages.microsoft.com/debian/$VERSION_ID/prod $VERSION_CODENAME main" \
          > /etc/apt/sources.list.d/mssql-release.list; \
    fi; \
    apt-get update; \
    ACCEPT_EULA=Y apt-get install -y msodbcsql18 mssql-tools18; \
    echo 'export PATH="$PATH:/opt/mssql-tools18/bin"' >> /etc/profile; \
    apt-get install -y --no-install-recommends unixodbc-dev; \
    pecl install sqlsrv-5.12.0 pdo_sqlsrv-5.12.0; \
    docker-php-ext-enable sqlsrv pdo_sqlsrv; \
    rm -rf /var/lib/apt/lists/*

# Composer (copiado da imagem oficial)
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Apache: módulos úteis
RUN a2enmod rewrite headers

# (mantido aqui para robustez, mas o vhost mapeado já resolve)
ENV APACHE_DOCUMENT_ROOT=/var/www/html/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' \
      /etc/apache2/sites-available/*.conf /etc/apache2/apache2.conf || true

# Copiar php.ini customizado
COPY ./.docker/php/php.ini /usr/local/etc/php/php.ini

# Diretório da aplicação
WORKDIR /var/www/html

# Apache + queue worker (billing-profit) no mesmo container
COPY .docker/php/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
RUN sed -i 's/\r$//' /usr/local/bin/docker-entrypoint.sh \
    && chmod +x /usr/local/bin/docker-entrypoint.sh

EXPOSE 80

ENTRYPOINT ["/bin/bash", "/usr/local/bin/docker-entrypoint.sh"]
