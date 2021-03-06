# traefik-proxy-boilerplate
No more messing around with nginx, apache config or certbot etc. This repository contains code to help quickly set up traefik as reverse proxy using docker compose or kubernetes cluster to host a website with SSL certificates using letsencrypt that auto renew. Hit letsencrypt api limits no problem this works with zerossl. 

# Traefik + Docker compose
![image](https://user-images.githubusercontent.com/11187601/148925399-2de1e90c-3e5d-4fe8-b46b-bd30c2cb3521.png)

# Directory structure

The project is divided into 2 main sections, first section contains code for docker compose config the second section contains code for kubernetes config yaml files.
```
.
├── LICENSE
├── README.md
├── docker                             # Docker Compose config
│   ├── docker-compose.yml
│   └── traefik.yml
└── kubernetes                         # Kubernetes config
    ├── kind                        
    │   └── kind.config.yaml           # Kind cluster config
    ├── metal-lb
    │   └── metallb-configmap.yaml     # metallb config (optional)
    ├── traefik                     
    │   ├── helm                    
    │   │   └── traefik.values.yaml    # Values to initialise helm chart
    │   ├── TLSoptions.yaml            # Traefik CRD TLSoptions config
    │   ├── middleware.yaml            # Traefik CRD middleware config
    │   └── traefik
    └── web-deployment
        ├── nginx-deployment.yaml      # Sample web deployment
        ├── nginx-service.yaml.        # Sample web service
        └── traefik-ingress-route.yaml # Treafik ingress route to webservice
```

## Steps to get Started 

It is assumed that you have created an cloud instance and installed docker and docker compose.

Installation instructions for Docker and Docker compose can be found here
https://docs.docker.com/engine/install/
https://docs.docker.com/compose/install/

Next you will need to set DNS A records that point to the Pubic IP of your instance that you will be using to host the website.
More info on creating A record can be found here https://www.namecheap.com/support/knowledgebase/article.aspx/319/2237/how-can-i-set-up-an-a-address-record-for-my-domain/
Also open port 80 and 443 in the security group of your instance that the instance is accessible from the internet.

Assuming that you have a working docker image for the website, update the docker compose file.
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

# Traefik + kind Kubernetes cluster
![image](https://user-images.githubusercontent.com/11187601/148925681-c634498c-e139-481a-b091-267920e3a7f8.png)
This guide assumes that you have limited computing resources and therefore we use kind to deploy a kubernetes cluster. If you have access to a fully functional kubernetes cluster, skip the kind installation and bootstraping steps. [start here ↓  ](#helm-installation)

Note : To start with you will need to set DNS A records that point to the Pubic IP of your instance that you will be using to host the website. More info on creating A record can be found here https://www.namecheap.com/support/knowledgebase/article.aspx/319/2237/how-can-i-set-up-an-a-address-record-for-my-domain/ Also open port 80 and 443 in the security group of your instance that the instance is accessible from the internet.

Let's first install kind, here is a link to quick start guide: https://kind.sigs.k8s.io/docs/user/quick-start/

Now that we have kind install lets create the kubernetes cluster
```
kind create cluster --config=kubernetes/kind/kind.config.yaml
```

## Helm installation
The installation instructions can be found here https://helm.sh/docs/intro/install/
below is a handly script published that can be found in the helm official docs
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

Now lets install traefik using  helm
```
helm install traefik traefik/traefik -n traefik --create-namespace -f kubernetes/traefik/helm/traefik.values.yaml
```

Next step is to configure traefik TLS Options and middleware
```
kubectl apply -f kubernetes/traefik/
```

Now lets create a sample kubernetes deployment and create a ingress route for it
```
kubectl apply -f kubernetes/web-deployment/
```


![image](https://user-images.githubusercontent.com/11187601/148923223-3e5820d8-1dfd-4f2f-b06f-df869aef7d4c.png)



