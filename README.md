## The Træfik Quickstart (Using Docker)

In this quickstart, we'll use [Docker compose](https://docs.docker.com/compose) to create our demo infrastructure.

To save you some time, you can checkout the quickstart from the [containous/traefik-quickstart](https://github.com/containous/traefik-quickstart) repository and read along.

### 1 — Launch Træfik — Tell It to Listen to Docker

First, create the `traefik-quickstart/traefik` folder. In this folder, we will create a `docker-compose.yml` file where we will define a service `reverse-proxy` that uses the official Træfik image.

`traefik-quickstart/traefik/docker-compose.yml`: 
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

**That's it. Now you can launch Træfik!**

In the `traefik-quickstart` folder, run the following command (that asks docker to deploy the containers defined in our `traefik/docker-compose.yml` file):

```shell
docker-compose -f traefik/docker-compose.yml up -d
```

If you'd like, you can open a browser and go to [http://localhost:8080](http://localhost:8080) to see Træfik's dashboard (we'll go back there once we'll have launched a service in step 2).

## 2 — Launch a Service — Træfik Detects It and Creates a Route for You 

Now that we have a Træfik instance up and running, we will deploy new services. 

Create the `traefik-quickstart/services/docker-compose.yml` file. There, we will define a new service (`whoami`) that is a simple webservice that outputs information about the machine where it is deployed (its IP adress, host, and so on):

`traefik-quickstart/services/docker-compose.yml`: 
```yaml
version: '3'

services:
  whoami:
    image: emilevauge/whoami #A container that exposes an API to show it's IP address
    labels:
      - "traefik.frontend.rule=Host:whoami.docker.localhost"

#we're deploying the services on the same network so that the containers can talk to each other
networks: 
   default: 
      external:
         name: traefik_default 
```

In the `traefik-quickstart` folder, run the following command to deploy your new container:
 
```shell
docker-compose -f services/docker-compose up -d
```

Go back to your browser ([http://localhost:8080](http://localhost:8080)) and see that Træfik has automatically detected the new container and updated its own configuration.

Now that Traefik has detected the service, a route is available and you can call your service! (Here, we're using curl)

```shell
curl -H Host:whoami.docker.localhost http://127.0.0.1
```

_Shows the following output:_
```yaml
I'm hosted on ef194d07634a
```

## 3 — Launch More Instances — Traefik Load Balances Them

In the `traefik-quickstart` folder, run the following command to start more instances of your `whoami` services:
 
```shell
docker-compose -f services/docker-compose.yml up --scale whoami=2 -d
```

Go back to your browser ([http://localhost:8080](http://localhost:8080)) and see that Træfik has automatically detected the new instance of the container.

Finally, see that Træfik load-balances between the two instances of your services by running the following command multiple times:

```shell
curl -H Host:whoami.docker.localhost http://127.0.0.1?hostedononly
```

The output will show alternatively one of the followings:

```yaml
I'm hosted on ef194d07634a
```

```yaml
I'm hosted on 6c3c5df0c79a
```

## 4 — Enjoy Træfik's Magic

Now that you have a basic understanding of how Træfik can automatically create the routes to your services and load balance them, it might be time to dive into [the documentation](https://docs.traefik.io/) and let Træfik work for you! Whatever your infrastructure is, there is probably [an available Træfik backend](https://docs.traefik.io/configuration/backends/available) that will do the job. 

Our recommandation would be to see for yourself how simple it is to enable HTTPS with [Træfik's let's encrypt integration](https://docs.traefik.io/user-guide/examples/#lets-encrypt-support) using the dedicated [user guide](https://docs.traefik.io/user-guide/docker-and-lets-encrypt/).