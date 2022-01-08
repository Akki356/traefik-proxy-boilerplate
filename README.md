# traefik-poxy-boilerplate
This repository contains code to help quickly set up traefik as reverse proxy using docker compose to host a website with ssl certificates using letsencrypt that auto renew. No more messing around with nginx, apache config or certbot etc. Hit letsencrypt api limits no problem this works with zerossl

# Steps to get Started 

It is assumed that you have install docker and docker compose.

Installation instructions for Docker and Docker compose can be found here
https://docs.docker.com/engine/install/
https://docs.docker.com/compose/install/

Also set DNS A records that point the pubic Ip of your instance that you will using to host this website.

Assuming that you have a working docker image of the website, update the docker compose file.
1. Update the image in the website section

``` 
website:
  image: example-image
```

2. Update the domain name at traefik.http.routers.blog.rule=Host(`example.com`)

```
    labels:
      - traefik.enable=true
      - traefik.http.routers.blog.rule=Host(`example.com`)
      - traefik.http.routers.blog.tls=true
      - traefik.http.routers.blog.tls.certresolver=myresolver
      
```

Now it time to update the traefik.yml
1. update the certificatesResolvers section with a valid email address

```
certificatesResolvers:
  myresolver:
    acme:
      email: email@example.com
      storage: acme.json
      # Uncomment the caServer to use the staging server inorder to avoid hitting letsencrypt api limits
      #caServer: "https://acme-staging-v02.api.letsencrypt.org/directory"
      httpChallenge:
        # used during the challenge
        entryPoint: web

```
2. Upate the rule in router section with the correct domain name.
```
http:
  routers:
    Router-1:
      service: "service-1"
      middlewares:
        - "secure-headers"
      # will terminate the TLS request
      rule: "Host(`example.com`)"
      tls:
        options: foo
        
 ```
 
 Start the containers using the below command.
 
 ```
 docker-compose up -d
 
 ```
 
 ## Change acme to zero ssl
 
 Signup for the free plan and generate the kid and hmacEncoded values in the developer section.
 https://app.zerossl.com/developer
 
 ```
 certificatesResolvers:
  myresolver:
    acme:
      email: email@example.com
      storage: acme.json
      caServer: "https://acme.zerossl.com/v2/DV90"
      eab:
        kid: jerlkqwejrlkjoij
        hmacEncoded: hqwkjeqkwejr
 ```

