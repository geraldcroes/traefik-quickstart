## The Traefik Quickstart (Using Docker)

### 1 -- Launch Traefik and Tell It to Listen to Docker

In this quickstart, we'll use [Docker compose](https://docs.docker.com/compose) to create our demo infrastructure.

First, we will define a service (`reverse-proxy`) that uses the official Traefik image.

Create the `traefik-quickstart/traefik/docker-compose.yml` file with the following content: 

```yaml
version: '3'

services:
  reverse-proxy:
    image: traefik #the official traefik docker image
    command: --api --docker --logLevel=DEBUG
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

A bit of explanations on the command line :
- `--api` enables the Web UI
- `--docker` tells Traefik to listens to Docker
- `--logLevel=DEBUG` even if we're sure you've figured that one out, it tells Traefik to be verbose in the logs

**Now you can launch Traefik!**

In the `traefik-quickstart/traefik` folder, run the following command:

```shell
docker-compose up -d
```

Open [http://localhost:8080](http://localhost:8080) in a browser to see Træfik's dashboard (we'll go back there once we'll have launched a service in step 2).

## 2 -- Launch a Service and See How Traefik Detects It and Creates a Route for You 

Create the `traefik-quickstart/services/docker-compose.yml` file with the following content:

```yaml
version: '3'
       
services:
 whoami:
   image: emilevauge/whoami #A container that exposes an API to show it's IP address
   labels:
     - "traefik.frontend.rule=Host:whoami.docker.localhost"
```

In the `traefik-quickstart/services` folder, run the following command to start your new container:
 
```shell
docker-compose up -d
```

Go back to your browser ([http://localhost:8080](http://localhost:8080)) and see that Traefik has automatically detected the new container and updated its own configuration.

And of course, don't forget to call your service! (Here, we're using curl)

```shell
curl -H Host:whoami.docker.localhost http://127.0.0.1
```

```yaml
Hostname: ef194d07634a
IP: 127.0.0.1
IP: ::1
IP: 172.17.0.4
IP: fe80::42:acff:fe11:4
GET / HTTP/1.1
Host: 172.17.0.4:80
User-Agent: curl/7.35.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 172.17.0.1
X-Forwarded-Host: 172.17.0.4:80
X-Forwarded-Proto: http
X-Forwarded-Server: dbb60406010d
```

## 3 -- Launch More Instances of your Services to Play With Traefik's Load Balancing Capabilities

In the `traefik-quickstart/services` folder, run the following command to start more instances of your service:
 
```shell
docker-compose up --scale whoami=2 -d
```

Go back to your browser ([http://localhost:8080](http://localhost:8080)) and see that Traefik has automatically detected the new instance of the container.

Finally, see that Traefik load-balances between the two instances of your services by running the following command multiple times:

```shell
curl -H Host:whoami.docker.localhost http://127.0.0.1
```

The output will show alternatively one of the followings:

```yaml
Hostname: ef194d07634a
IP: 127.0.0.1
IP: ::1
IP: 172.17.0.4
IP: fe80::42:acff:fe11:4
GET / HTTP/1.1
Host: 172.17.0.4:80
User-Agent: curl/7.35.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 172.17.0.1
X-Forwarded-Host: 172.17.0.4:80
X-Forwarded-Proto: http
X-Forwarded-Server: dbb60406010d
```

```yaml
Hostname: 6c3c5df0c79a
IP: 127.0.0.1
IP: ::1
IP: 172.17.0.3
IP: fe80::42:acff:fe11:3
GET / HTTP/1.1
Host: 172.17.0.3:80
User-Agent: curl/7.35.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 172.17.0.1
X-Forwarded-Host: 172.17.0.3:80
X-Forwarded-Proto: http
X-Forwarded-Server: dbb60406010d
```