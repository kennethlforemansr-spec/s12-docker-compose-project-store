# Student Project: Deploy the Retail Store Application with Docker Compose

## Project Goal

Your task is to containerize and deploy the AWS Containers Retail Sample application locally using Docker Compose.

You are given the application source code repository, but you are not given a completed `docker-compose.yml` file. You must study the service source folders, build the required Docker images, define the service dependencies, configure the correct environment variables, and run the full application stack on your machine.

This project is a practical DevOps exercise. You will work like a junior DevOps engineer who receives an application codebase and must prepare a repeatable local deployment for a multi-service application.

At the end of the project, the retail store frontend must be available at:

```text
http://localhost:8888
```

## Learning Objectives

By completing this project, you will practice:

- Linux command-line navigation, file inspection, and troubleshooting.
- Git workflow with feature branches, commits, and clean working-tree habits.
- GitHub collaboration by pushing your work to a remote repository.
- Docker image builds from real application source code.
- Dockerfile usage across Java, Go, Node.js, and Nginx services.
- Docker Compose service definitions, networking, ports, environment variables, and dependencies.
- Debugging container startup issues with `docker compose ps`, `docker compose logs`, and `curl`.
- Understanding how microservices communicate inside a Docker Compose network.
- Using AI tools responsibly to study a codebase, ask better technical questions, and understand implementation choices.

This project will be reviewed and worked through in class. You are still expected to try it yourself first, document your work, and bring clear questions when you get stuck.

## Using AI Tools

You are encouraged to use AI tools such as Claude, Codex, ChatGPT, or similar assistants to help you study the codebase, understand the services, create your Docker Compose plan, troubleshoot errors, and complete the project.

Do not use AI only to generate files without understanding them. When you use AI, ask it to explain what it is doing and why. Ask it to teach you the purpose of each service, each environment variable, each Dockerfile, and each Docker Compose section. You should be able to explain your final solution in your own words.

Good AI prompts for this project include:

- `Study this repository and explain the services I need to deploy with Docker Compose.`
- `Explain this Dockerfile line by line.`
- `Help me understand why this service needs this environment variable.`
- `Review my docker-compose.yml and tell me what might fail.`
- `Explain this Docker Compose error and teach me how to debug it.`

## Student Challenge

The first 10 S11/S12 students who complete this project successfully will receive `$100` each from DevOps Easy Learning.

## Due Date

This project is due on Sunday, July 05, 2026 before class.

Submit your work before class starts. Late or incomplete submissions may not qualify for the student challenge.

## Application Overview

The application is a microservices-based retail store. It contains a frontend UI, backend API services, static assets, and several backing services for persistence and messaging.

| Service | Purpose | Language / Runtime | Internal Port |
|---|---|---:|---:|
| `ui` | Web frontend and backend aggregator | Java 17 / Spring Boot | `8080` |
| `catalog` | Product catalog API | Go | `8080` |
| `carts` | Shopping cart API | Java 17 / Spring Boot | `8080` |
| `orders` | Orders API | Java 17 / Spring Boot | `8080` |
| `checkout` | Checkout orchestration API | Node.js / NestJS | `8080` |
| `assets` | Static product image and CSS assets | Nginx | `8080` |
| `catalog-db` | Catalog database | MariaDB | `3306` |
| `orders-db` | Orders database | MariaDB | `3306` |
| `carts-db` | Cart database | DynamoDB Local | `8000` |
| `checkout-redis` | Checkout session store | Redis | `6379` |
| `rabbitmq` | Orders messaging broker | RabbitMQ | `5672`, `15672` |

## Service Diagram

The diagram below shows the main runtime architecture. The browser only connects to the `ui` service through the host port `8888`. All other application calls happen inside the Docker Compose network.

```mermaid
flowchart LR
    browser[Browser<br/>localhost:8888] --> ui[ui<br/>Java Spring Boot<br/>:8080]

    ui --> catalog[catalog<br/>Go API<br/>:8080]
    ui --> carts[carts<br/>Java Spring Boot<br/>:8080]
    ui --> orders[orders<br/>Java Spring Boot<br/>:8080]
    ui --> checkout[checkout<br/>Node.js NestJS<br/>:8080]
    ui --> assets[assets<br/>Nginx static files<br/>:8080]

    catalog --> catalogDb[(catalog-db<br/>MariaDB<br/>:3306)]
    carts --> cartsDb[(carts-db<br/>DynamoDB Local<br/>:8000)]
    orders --> ordersDb[(orders-db<br/>MariaDB<br/>:3306)]
    orders --> rabbitmq[rabbitmq<br/>RabbitMQ<br/>:5672]
    checkout --> checkoutRedis[(checkout-redis<br/>Redis<br/>:6379)]
    checkout --> orders

    rabbitUi[Optional RabbitMQ UI<br/>localhost:15672] --> rabbitmq
```

## Dependency Graph

Use this graph to reason about `depends_on`, startup order, health checks, and restart behavior.

```mermaid
graph TD
    catalogDb[catalog-db] --> catalog[catalog]
    cartsDb[carts-db] --> carts[carts]
    ordersDb[orders-db] --> orders[orders]
    rabbitmq[rabbitmq] --> orders
    checkoutRedis[checkout-redis] --> checkout[checkout]
    orders --> checkout
    catalog --> ui[ui]
    carts --> ui
    orders --> ui
    checkout --> ui
    assets[assets] --> ui
```

## Git Workflow

Do not commit your work directly on the `main` branch.

Create a feature branch before making changes. Use this preferred naming pattern:

```text
feature/<your-name>-docker-compose-project
```

Example:

```bash
git checkout -b feature/jane-doe-docker-compose-project
```

Commit your work on your feature branch and push that branch when you are ready to submit.

Create or update a `README.md` file in your own feature branch with your deployment notes, screenshots, and final project evidence.

If you have any issue with the project, Docker, Git, or the deployment process, contact the Catch Up instructor before submitting.

## Required Deliverables

Submit the following:

1. A working `docker-compose.yml` file created by you.
2. Any Dockerfiles you create or modify.
3. A project `README.md` in your own branch explaining:
   - how to build the images,
   - how to start the application,
   - how to stop and remove the containers,
   - which URL to open in the browser on the school server,
   - any problems you encountered and how you solved them.
4. Screenshots in your branch showing:
   - your terminal with the application running,
   - `docker compose ps`,
   - the website working in a browser,
   - successful access to the homepage,
   - successful access to the catalog page.
5. Deploy the application on the school server and include the working server URL in your `README.md`.

## Important Rule

Do not use prebuilt application images for the main application services.

You must build these services from the source code in this repository:

```text
ui
catalog
carts
orders
checkout
assets
```

You may use public images for the backing services:

```text
mariadb:10.9
amazon/dynamodb-local:1.20.0
redis:6-alpine
rabbitmq:3-management
```

## Docker Image Build Requirements

The repository already contains reusable Dockerfiles for several runtimes. Use the correct build context and build arguments for each service.

| Service | Build Context | Dockerfile | Required Build Argument |
|---|---|---|---|
| `ui` | `src/ui` | `images/java17/Dockerfile` | `JAR_PATH=target/ui-0.0.1-SNAPSHOT.jar` |
| `catalog` | `src/catalog` | `images/go/Dockerfile` | `MAIN_PATH=main.go` |
| `carts` | `src/cart` | `images/java17/Dockerfile` | `JAR_PATH=target/carts-0.0.1-SNAPSHOT.jar` |
| `orders` | `src/orders` | `images/java17/Dockerfile` | `JAR_PATH=target/orders-0.0.1-SNAPSHOT.jar` |
| `checkout` | `src/checkout` | `images/nodejs/Dockerfile` | none |
| `assets` | `src/assets` | `src/assets/Dockerfile` | none |

You may choose your own local image names, but they should be clear and consistent. Example naming pattern:

```text
retail-store-ui:local
retail-store-catalog:local
retail-store-carts:local
retail-store-orders:local
retail-store-checkout:local
retail-store-assets:local
```

## Required Network Names

Your services must be able to reach one another by Docker Compose service name. Use these service names because the application environment variables below depend on them:

```text
ui
catalog
carts
orders
checkout
assets
catalog-db
orders-db
carts-db
checkout-redis
rabbitmq
```

## Port Publishing Requirements

Only the services that need browser or host access should publish ports to your machine.

| Service | Host Port | Container Port | Purpose |
|---|---:|---:|---|
| `ui` | `8888` | `8080` | Retail store frontend |
| `rabbitmq` | `15672` | `15672` | RabbitMQ management UI, optional |
| `rabbitmq` | `5672` | `5672` | RabbitMQ broker, optional host access |

The other services can stay internal to the Compose network.

If port `8888` is already being used on the school server, choose an alternate available host port and document it in your `README.md`. For example, you may map a different host port to the UI container port `8080`.

Example:

```text
8890:8080
```

## Shared Variable

Use one shared database password variable for MariaDB-backed services:

```text
MYSQL_PASSWORD=choose-a-local-password
```

You may place it in a `.env` file or pass it from the shell when running Docker Compose.

Example `.env` content:

```text
MYSQL_PASSWORD=store-local-pass
```

Do not commit real production secrets.

## Service Environment Variables

### `ui`

The UI service aggregates calls to the backend APIs.

| Variable | Required Value |
|---|---|
| `JAVA_OPTS` | `-XX:MaxRAMPercentage=75.0 -Djava.security.egd=file:/dev/urandom` |
| `SERVER_TOMCAT_ACCESSLOG_ENABLED` | `true` |
| `ENDPOINTS_CATALOG` | `http://catalog:8080` |
| `ENDPOINTS_CARTS` | `http://carts:8080` |
| `ENDPOINTS_ORDERS` | `http://orders:8080` |
| `ENDPOINTS_CHECKOUT` | `http://checkout:8080` |
| `ENDPOINTS_ASSETS` | `http://assets:8080` |

Optional variables:

| Variable | Default | Description |
|---|---|---|
| `PORT` | `8080` | Internal server port |
| `ENDPOINTS_HTTP_KEEPALIVE` | `true` | Backend HTTP keepalive setting |
| `RETAIL_UI_BANNER` | empty | Banner text shown at top of UI |

### `catalog`

The catalog service connects to MariaDB.

| Variable | Required Value |
|---|---|
| `GIN_MODE` | `release` |
| `DB_ENDPOINT` | `catalog-db:3306` |
| `DB_NAME` | `sampledb` |
| `DB_USER` | `catalog_user` |
| `DB_PASSWORD` | `${MYSQL_PASSWORD}` |
| `DB_MIGRATE` | `true` |

Optional variables:

| Variable | Default | Description |
|---|---|---|
| `PORT` | `8080` | Internal server port |
| `DB_READ_ENDPOINT` | empty | Optional read database endpoint |
| `DB_CONNECT_TIMEOUT` | `5` | Database connection timeout in seconds |

### `catalog-db`

Use image:

```text
mariadb:10.9
```

| Variable | Required Value |
|---|---|
| `MYSQL_ROOT_PASSWORD` | `${MYSQL_PASSWORD}` |
| `MYSQL_DATABASE` | `sampledb` |
| `MYSQL_USER` | `catalog_user` |
| `MYSQL_PASSWORD` | `${MYSQL_PASSWORD}` |

### `carts`

The carts service uses DynamoDB Local.

| Variable | Required Value |
|---|---|
| `JAVA_OPTS` | `-XX:MaxRAMPercentage=75.0 -Djava.security.egd=file:/dev/urandom` |
| `SERVER_TOMCAT_ACCESSLOG_ENABLED` | `true` |
| `SPRING_PROFILES_ACTIVE` | `dynamodb` |
| `CARTS_DYNAMODB_ENDPOINT` | `http://carts-db:8000` |
| `CARTS_DYNAMODB_CREATETABLE` | `true` |
| `AWS_ACCESS_KEY_ID` | `key` |
| `AWS_SECRET_ACCESS_KEY` | `dummy` |

Optional variables:

| Variable | Default | Description |
|---|---|---|
| `PORT` | `8080` | Internal server port |
| `CARTS_DYNAMODB_TABLENAME` | `Items` | DynamoDB table name |

### `carts-db`

Use image:

```text
amazon/dynamodb-local:1.20.0
```

No environment variables are required for the basic local deployment.

### `orders`

The orders service uses MariaDB and RabbitMQ.

| Variable | Required Value |
|---|---|
| `JAVA_OPTS` | `-XX:MaxRAMPercentage=75.0 -Djava.security.egd=file:/dev/urandom` |
| `SERVER_TOMCAT_ACCESSLOG_ENABLED` | `true` |
| `SPRING_PROFILES_ACTIVE` | `mysql,rabbitmq` |
| `SPRING_DATASOURCE_WRITER_URL` | `jdbc:mariadb://orders-db:3306/orders` |
| `SPRING_DATASOURCE_WRITER_USERNAME` | `orders_user` |
| `SPRING_DATASOURCE_WRITER_PASSWORD` | `${MYSQL_PASSWORD}` |
| `SPRING_DATASOURCE_READER_URL` | `jdbc:mariadb://orders-db:3306/orders` |
| `SPRING_DATASOURCE_READER_USERNAME` | `orders_user` |
| `SPRING_DATASOURCE_READER_PASSWORD` | `${MYSQL_PASSWORD}` |
| `SPRING_RABBITMQ_HOST` | `rabbitmq` |

Optional variables:

| Variable | Default | Description |
|---|---|---|
| `PORT` | `8080` | Internal server port |

### `orders-db`

Use image:

```text
mariadb:10.9
```

| Variable | Required Value |
|---|---|
| `MYSQL_ROOT_PASSWORD` | `${MYSQL_PASSWORD}` |
| `MYSQL_DATABASE` | `orders` |
| `MYSQL_USER` | `orders_user` |
| `MYSQL_PASSWORD` | `${MYSQL_PASSWORD}` |

### `checkout`

The checkout service uses Redis and calls the orders service.

| Variable | Required Value |
|---|---|
| `REDIS_URL` | `redis://checkout-redis:6379` |
| `ENDPOINTS_ORDERS` | `http://orders:8080` |

Optional variables:

| Variable | Default | Description |
|---|---|---|
| `PORT` | `8080` | Internal server port |
| `REDIS_READER_URL` | empty | Optional read Redis endpoint |

### `checkout-redis`

Use image:

```text
redis:6-alpine
```

No environment variables are required for the basic local deployment.

### `assets`

The assets service serves static files through Nginx.

| Variable | Required Value |
|---|---|
| `PORT` | `8080` |

### `rabbitmq`

Use image:

```text
rabbitmq:3-management
```

No environment variables are required for the basic local deployment.

The management UI is available on port `15672` if you publish it. The default local RabbitMQ credentials for this image are usually:

```text
guest / guest
```

## Suggested Startup Order

Docker Compose can start services together, but your application services depend on databases and message brokers. Configure dependencies thoughtfully.

Recommended dependency relationships:

| Service | Depends On |
|---|---|
| `catalog` | `catalog-db` |
| `carts` | `carts-db` |
| `orders` | `orders-db`, `rabbitmq` |
| `checkout` | `checkout-redis`, `orders` |
| `ui` | `catalog`, `carts`, `orders`, `checkout`, `assets` |

Some services may start before databases are fully ready. Your solution should tolerate this by allowing containers to restart, using health checks, or both.

## Validation Checklist

After starting the stack, run:

```bash
docker compose ps
```

All services should be running.

Then test the UI:

```bash
curl -L http://localhost:8888/
curl -L http://localhost:8888/catalog
curl -L http://localhost:8888/cart
```

Expected result:

```text
HTTP 200 responses after redirects
```

You should also open this URL in a browser:

```text
http://localhost:8888
```

On the school server, replace `localhost` with the server hostname or IP address and use your assigned or selected host port.

The homepage should show retail products. The catalog page should list products. The cart page should load successfully.

## Troubleshooting Hints

If the UI loads but products do not appear, check:

```bash
docker compose logs catalog
docker compose logs catalog-db
```

If cart actions fail, check:

```bash
docker compose logs carts
docker compose logs carts-db
```

If checkout fails, check:

```bash
docker compose logs checkout
docker compose logs orders
docker compose logs rabbitmq
```

If a Java service exits immediately, verify:

```text
SPRING_PROFILES_ACTIVE
database URL
database username
database password
RabbitMQ host
```

If a service cannot reach another service, verify that:

1. The service names match the names listed in this document.
2. The services are on the same Docker Compose network.
3. You are using internal container ports, not host ports, for service-to-service communication.

For example, the UI should call:

```text
http://catalog:8080
```

not:

```text
http://localhost:8080
```

Inside Docker Compose, `localhost` means the current container, not your laptop.

## Instructor Note

Before sharing the repository with students, make sure completed Compose solutions are not included in the student branch. In particular, remove or hide any existing `docker-compose.yml` files that already solve the deployment.

The students should receive the source code, Dockerfiles, and this project document, but not a finished Compose file.
