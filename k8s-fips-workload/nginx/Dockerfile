# Build ubuntu-focal-fips:latest image first
FROM ubuntu-focal-fips:latest

# Nginx
RUN apt-get update && apt-get install -y nginx

# Nginx: SSL
ADD server.conf /etc/nginx/conf.d/default.conf
RUN mkdir -p /etc/nginx/ssl
RUN openssl req -newkey rsa:2048 -subj "/C=GB/ST=London/L=London/O=Global Security/OU=IT Department/CN=example.com" -nodes -keyout /etc/nginx/ssl/test.key -x509 -days 365 -out /etc/nginx/ssl/test.crt

EXPOSE 443
STOPSIGNAL SIGTERM
CMD ["nginx", "-g", "daemon off;"]

