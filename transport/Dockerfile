FROM php:8.1-apache

RUN docker-php-ext-install mysqli

RUN a2enmod rewrite

COPY transport/ /var/www/html/

RUN chown -R www-data:www-data /var/www/html

COPY wait-for-it.sh /usr/local/bin/wait-for-it.sh
RUN chmod +x /usr/local/bin/wait-for-it.sh

EXPOSE 80

