FROM nginx:stable-alpine

EXPOSE 80 443
RUN apk add --no-cache bash vim nano apache2-utils

# install certbot
RUN apk add --no-cache certbot certbot-nginx
RUN ln -s $(which certbot) /usr/bin/certbot-auto
