FROM php:apache
RUN docker-php-ext-install pdo_mysql
COPY . /var/www/html/
COPY ./apache2.conf /etc/apache2/apache2.conf
RUN apt-get update && apt-get install -y curl
RUN a2enmod headers
RUN a2enmod rewrite


