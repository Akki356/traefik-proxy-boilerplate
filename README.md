# traefik-poxy-boilerplate
This repository contains code to help quickly set up traefik as reverse proxy using docker compose.

# Steps to get Started 

It is assumed that you have install docker and docker compose.

Installation instructions for Docker and Docker compose can be found here
https://docs.docker.com/engine/install/
https://docs.docker.com/compose/install/


Assuming that you have a working docker image of the website, update the docker compose file.
1. Update the image in the website section

``` 
website:
  image: example-image
```

2. Update the domain name at traefik.http.routers.blog.rule=Host(`example.com`)

```    labels:
      - traefik.enable=true
      - traefik.http.routers.blog.rule=Host(`example.com`)
      - traefik.http.routers.blog.tls=true
      - traefik.http.routers.blog.tls.certresolver=myresolver```


