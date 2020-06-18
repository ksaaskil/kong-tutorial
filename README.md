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
# Add service pointing to local my-service-1 (upstream service)
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
# Disable the plugin by first finding the plugin ID
$ curl http://localhost:8001/services/my_service_1/plugins/
# Then disable the plugin with the ID you found:
$ curl -X PATCH http://localhost:8001/services/my_service_1/plugins/{PLUGIN_ID} \
 --data "enabled=false"
```

### JWT

```bash
export KONG_SERVICE=my_service_1
$ curl -X POST http://localhost:8001/services/${KONG_SERVICE}/plugins \
    --data "name=jwt"
# Configure on route:
$ curl -X POST http://localhost:8001/routes/my-service-1/plugins \
    --data "name=jwt"
```

Add a consumer:

```bash
$ curl -X POST -d "username=user123&custom_id=SOME_CUSTOM_ID" http://localhost:8001/consumers/
```

Provision credential:

```bash
$ curl -X POST http://localhost:8001/consumers/user123/jwt -H "Content-Type: application/x-www-form-urlencoded"
```

In my case, I got `"key": "dym9zREuneIvdDRWhrCyGXonCxEe35NP"` and `"secret": "DBy5j4nQmxmkTFvTcROuklzZkAccOZYw"`.

List credentials:

```bash
$ curl http://kong:8001/consumers/user123/jwt
```

To craft JWT, the headers must be

```json
{
  "typ": "JWT",
  "alg": "HS256",
  "iss": "dym9zREuneIvdDRWhrCyGXonCxEe35NP"
}
```

Create a token at [https://jwt.io](https://jwt.io) with the credential key in `iss` and the credential secret for signing.

Once you have a token, issue a request with `Authorization` header:

```bash
$ curl http://localhost:8000/my-service-1 \
    -H "Authorization: Bearer ${TOKEN}"
```

If you have configured checking cookies with `config.cookie_names`:

```bash
$ curl --cookie jwt=${TOKEN} http://localhost:8000/my-service-1
```

## Misc

Execute commands in a running container:

```bash
# Check configuration is valid
$ docker exec -it kong-tutorial-kong kong check /etc/kong/kong.conf
```
