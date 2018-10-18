# Spring Demo

This application is intended to demonstrate some common components and practices that Uken's Platform Team employ when developing applications. Some features shown here are:
- Our common parent POM including:
  - Datadog APM and custom metrics
  - Common Spring / Spring boot starters
  - a code formatter
- MySQL database adapter, including Flyway DB for running migrations
- Interaction with Redis
- The Thymeleaf templating engine & how to serve simple web pages
- Swagger API documentation
- Docker as a development tool

## How to run it

The simplest way to run this application is by using docker. Here, we'll have you build an image, and run a container and it's dependencies using `docker-compose`. First, you'll need to install Java and Maven to build the code, and Docker to build the final image and run it.

Build the application with:
```shell
mvn clean package
```

Run the application with:
```shell
docker-compose up
```

If you are rebuilding your application and need a new docker image made, then run `docker build -t <your-image-name> .`.

This will start the application, spin up a database and redis instance, run migrations, and start the web server. Once it's all up you should be able to visit [http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html). See some `curl` examples below if you're inclined to run the code in this project.

## Database Failover

The MySQL JDBC driver doesn't have support to handle AWS Aurora failover properly, so we should use the MariaDB driver instead.

Documentation is here: https://mariadb.com/kb/en/library/failover-and-high-availability-with-mariadb-connector-j/

The database settings should look like this:

```yaml
spring:
  datasource:
    url: jdbc:mysql:aurora://<aurora-cluster-endpoint>:3306/<database>?usePipelineAuth=false
    driver-class-name: org.mariadb.jdbc.Driver
    hikari:
      data-source-properties:
        usePipelineAuth: "false" # this option is not compatible with aurora and it's true by default
```

## Database Migrations

Database migrations are managed with FlywayDB, which has a maven plugin (installed here via the pom.xml file).

To run migrations:

```shell
mvn flyway:migrate
```

## Testing

The following will run all unit and integration tests, as well as check the code formatter. You'll notice that the `pom.xml` sets up some resources like a MySQL equivalent in-memory database called H2, and a redis instance that will automatically start and stop when running integration tests. So long as you have maven installed, you should be able to run tests without any other dependencies.

```shell
mvn clean verify
```

## Swagger API Documentation

We use swagger to programmatically document our APIs. While running the server, you can visit [http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html) to see it in action. Much of what Swagger picks up is automatic, but you'll also note some `@Api` related annotation which add additional information.

## Datadog

We use Datadog for our APM and custom application metrics. You largely get these for free (along with a YourKit profiler agent, by the way) from our parent docker image. A number of JVM and application level metrics are produced via our metrics library witch is included in the parent pom that this project inherits from. An example of how to implement your own custom metrics is provided in the `ExampleRestController`.

## Code Formatting

We use the Google Code Formatter. Builds will fail if your code doesn't match the formatter. Format your code before committing with:

```shell
mvn fmt:format
```

## API

The following are some example requests you can make:

```shell
curl localhost:8080/api/redis/set -H 'Content-Type: application/json' -d '{"key": "my_key", "value": "my_value"}'
OK
```
```shell
curl localhost:8080/api/redis/get/my_key
my_value
```

```shell
curl localhost:8080/api/users -d ''
{"id":"e5aad22d-cd08-4143-9a5b-11bc02887613"}
```

```shell
curl localhost:8080/api/users/e5aad22d-cd08-4143-9a5b-11bc02887613
{"id":"e5aad22d-cd08-4143-9a5b-11bc02887613"}
```

```shell
curl -XDELETE localhost:8080/api/users/e5aad22d-cd08-4143-9a5b-11bc02887613
{"id":"e5aad22d-cd08-4143-9a5b-11bc02887613"}
```

```shell
curl localhost:8080/api/users/e5aad22d-cd08-4143-9a5b-11bc02887613 | jq
{
  "timestamp": 1520545062775,
  "status": 404,
  "error": "Not Found",
  "exception": "com.uken.platform.demo.exceptions.UserNotFound",
  "message": "User 'e5aad22d-cd08-4143-9a5b-11bc02887613' could not be found",
  "path": "/api/users/e5aad22d-cd08-4143-9a5b-11bc02887613"
}
```
