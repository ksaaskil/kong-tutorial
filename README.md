# Kong tutorial

### Prerequisites

First, start [Docker Compose](./docker-compose.yaml):

```bash
$ docker-compose up
```

This starts Kong, Postgres for managing Kong state, and services used as targets.

Check Kong is available:

```bash
$ curl -i http://localhost:8001
```

### Creating a service

Add service pointing to local container running in Docker Compose:

```bash
# Add service pointing to local my-service-1 service
$ curl -i -X POST http://localhost:8001/services \
 --data name=my_service_1 \
 --data url='http://my-service-1'
# Check service configuration in Kong
$ curl -i http://localhost:8001/services/my_service_1
# Add route to service
$ curl -i -X POST http://localhost:8001/services/my_service_1/routes \
  --data 'paths[]=/my-service-1' \
  --data 'name=my-service-1'
```

Now navigate to [`http://localhost:8000/my-service-1`](http://localhost:8000/my-service-1) to see Nginx welcome page.

### Adding plugins

Add API key authentication plugin:

```bash
$ curl -i -X POST \
  --url http://localhost:8001/services/my_service_1/plugins/ \
  --data 'name=key-auth'
# Check authentication is working, you should get Unauthorized:
$ curl -i http://localhost:8000/my-service-1
```

### Creating consumers

Create consumers:

```bash
# Create consumer with name "Helen"
$ curl -i -X POST \
  --url http://localhost:8001/consumers/ \
  --data "username=Helen"
# Give the user a key
$ curl -i -X POST \
  --url http://localhost:8001/consumers/Helen/key-auth/ \
  --data 'key=SUPER_SECRET'
# Make request with the key:
$ curl -i http://localhost:8000/my-service-1 \
  --header "apikey: SUPER_SECRET"
```

## Misc

Execute commands in a running container:

```bash
# Check configuration is valid
$ docker exec -it kong-tutorial-kong kong check /etc/kong/kong.conf
```
