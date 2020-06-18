# Kong tutorial

First start Docker Compose:

```bash
$ docker-compose up
```

This starts Kong, Postgres for managing Kong state, and services used as targets.

Then issue following commands:

```bash
# Check Kong is available
$ curl -i http://localhost:8001
```

Add service pointing to `mockbin.org`:

```bash
# Add mock service pointing to mockbin.org
$ curl -i -X POST http://localhost:8001/services \
 --data name=example_service \
 --data url='http://mockbin.org'
# Check service exists
$ curl -i http://localhost:8001/services/example_service
# Add route
$ curl -i -X POST http://localhost:8001/services/example_service/routes \
  --data 'paths[]=/mock' \
  --data 'name=mocking'
# Check the route works as expected
$ curl -i http://localhost:8000/mock
# Add
```

Add service pointing to local container:

```bash
# Add service pointing to local API
$ curl -i -X POST http://localhost:8001/services \
 --data name=my_service_1 \
 --data url='http://my-service-1'
# Check service state
$ curl -i http://localhost:8001/services/my_service_1
# Add route
$ curl -i -X POST http://localhost:8001/services/my_service_1/routes \
  --data 'paths[]=/my-service-1' \
  --data 'name=my-service-1'
```

Now navigate to [`http://localhost:8000/my-service-1`](http://localhost:8000/my-service-1) to see Nginx welcome page.
