FROM php:8.2-fpm as builder

# Installation des dépendances système
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip \
    && rm -rf /var/lib/apt/lists/*

# Installation des extensions PHP
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Installation de Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Configuration du répertoire de travail
WORKDIR /var/www/html

# Copie des fichiers de dépendances
COPY composer.json composer.lock ./

# Installation des dépendances
RUN composer install --no-dev --no-scripts --no-autoloader --prefer-dist

# Copie du reste des fichiers
COPY . .

# Génération de l'autoloader
RUN composer dump-autoload --optimize

# Image finale
FROM php:8.2-fpm

# Copie des fichiers depuis le builder
COPY --from=builder /var/www/html /var/www/html
COPY --from=builder /usr/local/etc/php/conf.d /usr/local/etc/php/conf.d

# Installation des extensions PHP nécessaires
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    && docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd \
    && rm -rf /var/lib/apt/lists/*

# Configuration PHP
RUN echo "memory_limit=256M" > /usr/local/etc/php/conf.d/custom.ini && \
    echo "upload_max_filesize=20M" >> /usr/local/etc/php/conf.d/custom.ini && \
    echo "post_max_size=20M" >> /usr/local/etc/php/conf.d/custom.ini && \
    echo "max_execution_time=30" >> /usr/local/etc/php/conf.d/custom.ini && \
    echo "date.timezone=Europe/Paris" >> /usr/local/etc/php/conf.d/custom.ini

# Configuration des permissions
RUN chown -R www-data:www-data /var/www/html && \
    chmod -R 755 /var/www/html

# Définition de l'utilisateur
USER www-data 