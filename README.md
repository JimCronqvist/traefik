# A Traefik setup for local development
A simple traefik setup for local development. Traefik is a cloud-native reverse proxy and load balancer built for 
microservices and containerized workloads.

## Usage
Clone the repo:
```
git clone https://github.com/JimCronqvist/traefik.git
```
Start the container:
```
docker-compose up -d
```
Access your application(s) via **http** at: http://localhost:3000

Access your application(s) via **https** at: https://localhost:3443

Access the Traefik dashboard at: http://localhost:9000

## Configure your docker-compose services to use traefik
Add the following labels to **each of the services** in the docker-compose file
```
labels:
  traefik.enable: true
  traefik.http.routers.[name].rule: Host(`my-service.localhost`) && PathPrefix(`/`)

  # Optional labels - only apply if you need to.
  # traefik.http.routers.[name].priority: 0
  # traefik.http.services.[name].loadbalancer.server.port: 3000
```
Replace the [name] placeholder with the name of the service in docker-compose, and update the rule for your setup.
For rule syntax see https://doc.traefik.io/traefik/routing/routers/#rule

*Note: If your reason for setting the loadbalancer server port is because no default port is exposed in the container, 
you can also set the port in the docker-compose file under each service `expose: [3000/tcp]`, common for `node` apps.*

Add the following section to your docker-compose file at the end to connect all services to the traefik network
```
networks:
  default:
    name: traefik
    external: true
```

Note that for existing setups, you will need to recreate the services **once** for it to work, by running `docker-compose up --force-recreate`

## For HTTPS usage (optional) - one-time setup

### Install mkcert
```
# Ubuntu
sudo apt install -y libnss3-tools

# Mac
brew install mkcert

# Windows
choco install -y mkcert
```

### Generate the CA and the certificate
```
mkcert -install
mkcert -cert-file certs/cert.pem -key-file certs/key.pem localhost 127.0.0.1 ::1 *.localhost *.dev *.[company].dev
```

### Installing the root CA on other systems
Installing in the trust store does not require the CA key, so you can export the CA certificate and use mkcert to 
install it in other machines.

- Look for the rootCA.pem file in `mkcert -CAROOT`
- copy it to a different machine
- set `$CAROOT` to its directory
- run `mkcert -install`


### Installing the root CA on Mobile devices
See https://github.com/FiloSottile/mkcert?tab=readme-ov-file#mobile-devices
