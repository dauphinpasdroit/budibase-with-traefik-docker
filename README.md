# Budibase with Traefik

## Introduction

The idea behind this repository is to deploy a budibase instance with a reverse proxy (Treafik) and an automatically generated certificate, in order to expose over 443 and TLS.

The certificate will be automatically generated using Let's Encrypt.

I personnaly use a public domain name. 

## Installation step

1. Deploy a VM with your favorite Linux OS, install Docker Engine and install Docker compose.

2. Clone this repository ``git clone https://github.com/alphaxr6/budibase-with-traefik-docker.git``

3. You'll have to update the environment variable in the ``.env``.

Variables to setup:

- ``DOMAIN=`` ``SUBDOMAIN_TRAEFIK=``. Those variables will be use to setup the domain name on the certificate.
- ``CF_API_EMAIL=``. It will be use to generate the certificate as well.

And setup the database variables.

4. Then you can launch the docker compsoe ``docker compose up -d`` and wait for the instance to start.

5. Connect to your Budibase instance using the ``subdomain.domain`` you have setup in the ``.env`` file.

## Files explanation

### Docker compose

Basically, the ``docker-compose.yml`` file is the same that the one provided [here](https://docs.budibase.com/docs/docker-compose)

### Docker compose override

The Traefik container will be setup in the the ``docker-compose.override.yml`` file.

You'll have to add the label in the ``proxy-service``:

- ``traefik.enable: "true"`` : Enable the routing configuration for the containter.
- ``traefik.http.routers.budibase-http.entrypoints: "web"`` : Declare the entrypoint as 'web' (see the conf of the `traefik.yml`)
- ``traefik.http.routers.budibase-http.rule: "Host(`${SUBDOMAIN_TRAEFIK}.${DOMAIN}`)"`` : Declare the host name you're supposed to use to access the docker.
- ``traefik.http.routers.budibase-http.middlewares: "SslHeader@file"``
- ``traefik.http.routers.budibase-https.middlewares: "SslHeader@file"``
- ``traefik.http.routers.budibase-https.entrypoints: "websecure"`` : Declare the entrypoint as 'websecure' (see the conf of the `traefik.yml`)  
- ``traefik.http.routers.budibase-https.rule: "Host(`${SUBDOMAIN_TRAEFIK}.${DOMAIN}`)"`` : Declare the host name you're supposed to use to access the docker.
- ``traefik.http.routers.budibase-https.tls: "true"``: Enable the TLS. Use the `tls.yaml` to setup it.
- ``traefik.http.routers.budibase-https.tls.certresolver: "letsencrypt"`` : Declare the certificate resolver, here letsencrypt.
- ``traefik.http.services.budibase.loadbalancer.server.port: "${MAIN_PORT}"`` : Setup the port access of the docker. 

The Traefik container is setup with the following code:

```
  traefik:
    image: traefik:2.6
    restart: always
    ports:
      - 443:443
      - 80:80
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik.yaml:/traefik.yaml:ro
      - ./conf/:/etc/traefik/conf
      - ./shared/:/shared
    networks:
      budibase_net: {}

```

The volumes are setup to take the conf needed by traefik.

The `conf` folder, with the files `headers.yml`and `tls.yaml`, is the Traefic configuration, and also the `traefik.yaml` file.
