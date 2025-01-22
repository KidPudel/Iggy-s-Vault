Tools to manage multi-container applications on an individual server using yml config file
If load is too high, we might want to utilize [[Kubernetes]]

compose:
```yaml
services:
	frontend:
		images: example/webapp
		ports:
			- "443:8043"
		networks:
			- front-tier
			- back-tier
		...
	backend:
		images: example/database
		networks:
			- back-tier
		...
	database:
		images: example/
```

```yaml
version: "3"
services:
  payment-order-service:
    build: .
    environment:
      PAYMENT_ORDER_SERVICE_POSTGRES_ADDRESS: postgres://postgres:postgres@postgres:5432/postgres?sslmode=disable
      PAYMENT_ORDER_SERVICE_KAFKA_ADDRESS: http://kafka:9092
    ports:
      - 8080:8080
    volumes:
      - ./certs/kafka:/certs/kafka
      - ./certs/postgres:/certs/postgres
    depends_on:
      - kafka
      - postgres
    command:
      - ./wait-for-it.sh
      - kafka:9092
      - --
      - ./wait-for-it.sh
      - postgres:5432
      - --
      - payment-order-service

  kafka:
    image: bitnami/kafka:3.6.2
    environment:
      KAFKA_CFG_NODE_ID: 0
      KAFKA_CFG_PROCESS_ROLES: controller,broker
      KAFKA_CFG_LISTENERS: PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
      KAFKA_CFG_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,EXTERNAL://localhost:9094
      KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
      KAFKA_CFG_CONTROLLER_QUORUM_VOTERS: 0@kafka:9093
      KAFKA_CFG_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE: "true"
    ports:
      - 9094:9094

  kafka-ui:
    image: provectuslabs/kafka-ui:v0.7.2
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:9092
    ports:
      - 10100:8080
    depends_on:
      - kafka

  postgres:
    image: postgres:16.2-alpine3.19
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
    ports:
      - 5432:5432

  pgadmin:
    image: dpage/pgadmin4:8.5
    environment:
      PGADMIN_DEFAULT_EMAIL: postgres@example.com
      PGADMIN_DEFAULT_PASSWORD: postgres
      PGADMIN_DISABLE_POSTFIX: "true"
    ports:
      - 10200:80
    depends_on:
      - postgres

```

compose.deploy:
```yaml
version: "3.8"
services:
  payment-order-service:
    container_name: ${SERVICE_NAME}
    image: ${SERVICE_IMAGE}
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - 127.0.0.1:$PAYMENT_ORDER_SERVICE_SERVER_PORT:$PAYMENT_ORDER_SERVICE_SERVER_PORT
    networks:
      - service_network
    volumes:
      - /etc/ssl/certs/kafka:/certs/kafka
      - /etc/ssl/certs/postgres:/certs/postgres

networks:
  service_network:
    name: service_network
    external: true
```


commands:
- `docker-compose up`: run containers
- `docker-compose down`: stop containers



# Volumes
Way to share data between container and local host or multiple containers

mounting/mapping files with write capabilities
```yml
    volumes:
      - ./db/migrations:/service/db/migrations:rw # map with the host with read write mode

```


# Networking
By default compose sets up a single network for your app. Each container for a service joins the default network and is both reachable by other containers on that network, and discoverable by the service's name.
> each network is given a name on the "project name"

For example, suppose your app is in a directory called `myapp`, and your `compose.yml` looks like this:

```yaml
services:
  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres
    ports:
      - "8001:5432"
```

When you run `docker compose up`, the following happens:

1. A network called `myapp_default` is created.
2. A container is created using `web`'s configuration. It joins the network `myapp_default` under the name `web`.
3. A container is created using `db`'s configuration. It joins the network `myapp_default` under the name `db`.


- `docker-compose exec flyway-container-name flyway migrate`: `exec` allows to communicate with other containers defined in docker-compose.yml