## The Træfik Quickstart (Using Docker)

### 1 -- Launch Træfik and Tell It to Listen to Docker

In this quickstart, we'll use [Docker compose](https://docs.docker.com/compose) to create our demo infrastructure.

First, create the `traefik-quickstart` folder. In this folder, we will create a `traefik-docker-compose.yml` file where we will define a service `reverse-proxy` that uses the official Træfik image.

`traefik-quickstart/traefik-docker-compose.yml`: 
```yaml
version: '3'

services:
  reverse-proxy:
    image: traefik #The official Traefik docker image
    command: --api --docker
    ports:
      - "80:80"     #The HTTP port
      - "8080:8080" #The Web UI
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock #So that Traefik can listen to the Docker events
```

A bit of explanations on the command line :
- `--api` enables the Web UI
- `--docker` tells Træfik to listens to Docker (so that Træfik can automatically adapt its configuration)

**Now you can launch Træfik!**

In the `traefik-quickstart` folder, run the following command (that asks docker to deploy the containers defined in our `traefik-docker-compose.yml` file):

```shell
docker-compose -f traefik-docker-compose.yml up -d
```

Open [http://localhost:8080](http://localhost:8080) in a browser to see Træfik's dashboard (we'll go back there once we'll have launched a service in step 2).

## 2 -- Launch a Service and See How Træfik Detects It and Creates a Route for You 

Now that we have a Træfik instance up and running, we will deploy new services. 

Create the `traefik-quickstart/services-docker-compose.yml` file. There, we will define a new service (`whoami`) that is a simple webservice that outputs information about the machine where it is deployed (its IP adress, host, and so on):

```yaml
version: '3'
       
services:
 whoami:
   image: emilevauge/whoami #A container that exposes an API to show it's IP address
   labels:
     - "traefik.frontend.rule=Host:whoami.docker.localhost"
```

In the `traefik-quickstart` folder, run the following command to deploy your new container:
 
```shell
docker-compose -f services-docker-compose up -d
```

Go back to your browser ([http://localhost:8080](http://localhost:8080)) and see that Træfik has automatically detected the new container and updated its own configuration.

And of course, don't forget to call your service! (Here, we're using curl)

```shell
curl -H Host:whoami.docker.localhost http://127.0.0.1
```

_Shows the following output:_
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

In the `traefik-quickstart` folder, run the following command to start more instances of your `whoami` services:
 
```shell
docker-compose -f services-docker-compose.yml up --scale whoami=2 -d
```

Go back to your browser ([http://localhost:8080](http://localhost:8080)) and see that Træfik has automatically detected the new instance of the container.

Finally, see that Træfik load-balances between the two instances of your services by running the following command multiple times:

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