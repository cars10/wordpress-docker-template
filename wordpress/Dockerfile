FROM wordpress:6-fpm

RUN apt-get update && apt-get install -y vim nano && rm -rf /var/lib/apt/lists/*

# use PHP production config
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"
