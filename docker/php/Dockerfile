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
       pecl install sqlsrv pdo_sqlsrv; \
       docker-php-ext-enable sqlsrv pdo_sqlsrv; \
       rm -rf /var/lib/apt/lists/*

# Composer (copiado da imagem oficial)
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Apache: módulos úteis
RUN a2enmod rewrite headers

# Ajusta DocumentRoot para /var/www/html/public se não usar o vhost externo
# (mantido aqui para robustez, mas o vhost mapeado já resolve)
ENV APACHE_DOCUMENT_ROOT=/var/www/html/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' \
      /etc/apache2/sites-available/*.conf /etc/apache2/apache2.conf || true

# Diretório da aplicação
WORKDIR /var/www/html

# Permissões típicas de Laravel (se aplicável)
# RUN chown -R www-data:www-data /var/www/html \
#     && find storage bootstrap/cache -type d -exec chmod 775 {} \; || true

EXPOSE 80