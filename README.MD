# Wordpress Docker Template

## Overview
Services:

* `wordpress` container
    * php-fpm process that runs the wordpress php application. **NOT** exposed to the internet.
    * Custom image built via `./wordpress/Dockerfile` based on the official wordpress docker image
* `db` container
    * Database used for the wordpress application
    * Official mariadb docker image
* `nginx` container
    * Reverse proxy responsible for SSL and handling connections to the wordpress container.
    * Custom image built via `./nginx/Dockerfile` based on the official nginx docker image

See `docker compose.yml` for more details.

## Services

### Wordpress
We use a custom wordpress image based on the official wordpress fpm image. Because it only runs fpm it should **never** be directly exposed to the internet, **never** expose the ports of the wordpress service on our host! Only access it via our nginx container.

We use a custom image to
* Add dependencies via `apt`, currently only `vim` & `nano` for easier debugging. But many wordpress options/plugins require custom php packages, this is the place to add them.
* Overwrite the default `php.ini` with the provided config for production environments.

### Nginx
We use a custom nginx image to
* Add dependencies via `apk`, currently `bash`, `vim`, `nano` and `certbot`.

## Project setup
To setup the inital project:
1. Create your `.env` config file: `cp .env.example .env`
2. Adjust the setting in your `.env` file
3. Copy the `scripts/backup` script, look inside for instructions
4. Copy the `scripts/renew_ssl` script, look inside for instructions
3. Run `docker compose up -d`


## Project updating and maintenance
The project dependencies should be updated periodically. To do so:

### Wordpress
We can use the build-in updater in Wordpress.

### Nginx
1. Run `docker compose build nginx`, we automatically use the current stable version.
2. Run `docker compose up -d nginx`

### DB
1. Update the mariadb version in `docker compose.yml` at `db/image`
2. Run `docker compose up -d db`
3. If it works: commit your changes to the `docker compose.yml`!


## Configuration
All relevant config files are present on the host system and mounted into the containers via `volumes`.

### Wordpress
To change php settings adjust the custom php.ini in `./wordpress/php_custom.ini`. After changing restart the wordpress container via `docker compose restart wordpress`.

### Nginx
To change nginx settings adjust the config in `./nginx/nginx.conf`. To change the site specific settings use `./nginx/sites/worpress.conf`. After changing restart the nginx container via `docker compose restart nginx`.

## SSL

Generate SSL certificates:

```
docker compose down

# edit nginx/nginx.conf and remove the include at the end

docker compose up -d nginx
docker compose exec nginx certbot certonly --nginx -d example.com --email you@example.com --agree-tos --non-interactive --expand
docker compose down

# edit nginx/nginx.conf and add the include again

```

Certificates are automatically renewed weekly via `renew_ssl` script in `/etc/cron.weekly`. If needed you can renew manually like this:

```
docker compose exec nginx certbot renew
```

## Backup & Restore

The backup consists of two parts:

1. The mysql database
2. The wordpress data

### Backup

### Restore

Start by importing the database:

1. Stop the cluster: `docker compose down`
2. Start the db with `docker compose up db` and wait for `mysqld: ready for connection.`
3. Unzip the database backup that you want to use: `gunzip backups/2021_08_30-19_17_02/db.sql.gz`
4. Import the database backup: `docker exec -i db sh -c 'exec mysql -u$MYSQL_USER -p"$MYSQL_PASSWORD" $MYSQL_DATABASE' < backups/2021_08_30-19_17_02/db.sql`
5. Stop the cluster: `docker compose down`

Then extract the wordpress data:

1. Move possible old data dir: `sudo mv wordpress/data wordpress/data.old`
2. Unzip backup `unzip backups/2021_08_30-19_17_02/wordpress_data.zip`
3. Move the data to `worpress/data`

Then start the cluster `docker compose up -d`
