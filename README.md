# Docker Nginx Proxy

A simple example of how to use the Docker Nginx Proxy as a simple router that can connect to LetsEncrypt and provide free SSL certificate.

I used the following images and projects:
- Docker Nginx Proxy: https://github.com/nginx-proxy/nginx-proxy
- Docker LetsEncrypt Nginx Proxy Companion: https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion

See [docker-compose.yml](docker-compose.yml) file:
```yaml
version: "3.3"

services:
  whoami:
    container_name: whoami
    image: containous/whoami
    restart: unless-stopped
    network_mode: "bridge"
    environment:
      - VIRTUAL_HOST=whoami.localhost,whoami.joaogabriel.org
      - LETSENCRYPT_HOST=whoami.joaogabriel.org
      - LETSENCRYPT_EMAIL=gabriel@joaogabriel.org

  nginx_proxy:
    container_name: nginx_proxy
    image: jwilder/nginx-proxy:alpine
    volumes:
      - nginx_proxy_nginx_certs:/etc/nginx/certs
      - nginx_proxy_nginx_vhost_d:/etc/nginx/vhost.d
      - nginx_proxy_nginx_html:/usr/share/nginx/html
      - /var/run/docker.sock:/tmp/docker.sock:ro
    restart: unless-stopped
    network_mode: "bridge"
    environment:
      - ENABLE_IPV6=true
      - DEFAULT_HOST=whoami.joaogabriel.org
    ports:
      - 80:80
      - 443:443

  lets_encrypt:
    container_name: lets_encrypt
    image: jrcs/letsencrypt-nginx-proxy-companion
    volumes:
      - nginx_proxy_nginx_certs:/etc/nginx/certs
      - nginx_proxy_nginx_vhost_d:/etc/nginx/vhost.d
      - nginx_proxy_nginx_html:/usr/share/nginx/html
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - NGINX_PROXY_CONTAINER=nginx_proxy
    restart: unless-stopped
    network_mode: "bridge"
    depends_on:
      - nginx_proxy

volumes:
  nginx_proxy_nginx_certs:
  nginx_proxy_nginx_vhost_d:
  nginx_proxy_nginx_html:
``` 

The volumes are used to persist SSL certificates (do this because LetsEncrypt has a rate limit).

### Another option
Another option is to use Traefik, a more complete proxy with a dashboard included.
https://github.com/jgcl/docker-traefik-proxy