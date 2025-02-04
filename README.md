# CPE_DEVOPS

## TP1 - Docker & DevOps

### 📌 Commandes exécutées

```bash
docker build -t moha/db_postgres .
docker network create app-network
docker run -p 8888:5000 --net=app-network -d --name db_postgres moha/db_postgres
```

---

### ❓ 1-1 Why should we run the container with a flag `-e` to give the environment variables?

Using `-e` allows dynamic configuration without modifying the code. This keeps **configuration and code separate**, improving maintainability. It also facilitates **deployments across different environments** (dev, pre-prod, prod) without modifying the container.

---

### ❓ 1-2 Why do we need a volume to be attached to our Postgres container?

Without a volume, database data **disappears when the container is rebuilt**. A volume ensures **persistent storage**, allowing data to survive container restarts or rebuilds.

---

### 🔍 1-3 Document your database container essentials

#### **Database Commands**

```bash
docker run --net=app-network -d -v pg_data:/var/lib/postgresql/data --name db_postgres moha/db_postgres
```

#### **Database Dockerfile**

```Dockerfile
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd

COPY ./CreateScheme.sql /docker-entrypoint-initdb.d/
COPY ./InsertData.sql /docker-entrypoint-initdb.d/

VOLUME [ "data" ]
```

#### **Adminer Launcher** (for database UI management)

```bash
docker run \
    -p "8090:8080" \
    --net=app-network \
    --name=adminer \
    -d \
    adminer
```

---

## 🚀 Backend-API

#### **Compilation & Execution**

```bash
javac Main.java
docker build -t java-app .
docker run --rm java-app
```

#### **Backend Dockerfile**

```Dockerfile
# Stage 1: Build the Java application
FROM openjdk:17 AS builder

WORKDIR /usr/src/
COPY Main.java .

# Compile the Java file
RUN javac Main.java

# Stage 2: Run the compiled Java application
FROM openjdk:17
COPY --from=builder /usr/src/Main.class .
CMD ["java", "Main"]
```

---

### 🔄 1-5 Why do we need a reverse proxy?

A **reverse proxy** enhances security, performance, and scalability by acting as an intermediary between clients and backend services:

- **Security** – Hides backend services, preventing direct access.
- **Load Balancing** – Distributes requests efficiently across servers.
- **SSL Termination** – Manages HTTPS encryption for secure connections.
- **Caching** – Reduces backend load by storing frequent responses.
- **Routing** – Directs traffic to the correct backend service.

📌 **Example:** In this setup, Apache forwards client requests to `backend_app:8080`, keeping the backend isolated and secure.

---

### 🛠 1-6 Why is `docker-compose` so important?

`docker-compose` simplifies multi-container management by allowing you to **define and run all services with a single command**.

🔹 **Key Benefits:**

- 🏗 **Easier setup** – Define everything in a `docker-compose.yml` file.
- 🚀 **Faster deployment** – Start everything with `docker-compose up -d`.
- 🔄 **Automatic networking** – Services can talk to each other using their names.
- 🔧 **Scalability** – Easily scale services (`docker-compose up --scale backend=3`).
- 🔁 **Reproducibility** – Ensures the same setup runs everywhere.

---

### 📌 1-7 Most important `docker-compose` commands

|Command|Description|
|---|---|
|`docker-compose up -d`|Starts all services in the background.|
|`docker-compose down`|Stops and removes all services, networks, and volumes.|
|`docker-compose build`|Builds/Rebuilds the services from `docker-compose.yml`.|
|`docker-compose ps`|Lists running services.|
|`docker-compose logs -f`|Streams live logs of all services (useful for debugging).|
|`docker-compose exec <service> bash`|Opens a terminal inside a running container.|
|`docker-compose restart <service>`|Restarts a specific service.|
|`docker-compose rm -f`|Removes stopped containers.|
|`docker-compose stop`|Stops all running containers without removing them.|
|`docker-compose up --scale backend=3`|Runs 3 instances of the `backend` service.|

💡 **Pro tip:** Use `docker-compose down -v` for a full reset, including volume data.

---

### 📜 1-8 Documenting your `docker-compose.yml` file

```yml
version: '3.7'  # Docker Compose version

services:
    backend:
        build: ./Backend-API  # Builds backend from `Backend-API` directory
        container_name: backend_api  # Fixed name for easy reference
        networks:
            - app-network  # Connects to the shared network
        depends_on:
            - database  # Ensures database starts first
        ports:
          - "8080:8080"  # Maps container port 8080 to host port 8080

    database:
        image: postgres:14.1-alpine  # Lightweight Postgres image
        container_name: db_postgres  # Fixed name for database container
        environment:
          POSTGRES_DB: db  # Database name
          POSTGRES_USER: usr  # Username
          POSTGRES_PASSWORD: pwd  # Password
        ports:
          - "5433:5432"  # Maps Postgres port 5432 inside the container to 5433 on the host
        networks:
            - app-network  # Ensures connectivity with other services
        volumes:
            - pg_data:/var/lib/postgresql/data  # Persistent storage

    httpd:
        build: ./HTTP-server  # Builds the HTTP server from `HTTP-server`
        container_name: server  # Fixed name for clarity
        ports:
            - "80:80"  # Exposes on port 80 (accessible via `localhost`)
        networks:
            - app-network  # Links to backend and database
        depends_on:
            - backend  # Ensures backend is running first
            - database  # Ensures database is running first

networks:
    app-network:  # Defines the shared network

volumes:
    pg_data:  # Persistent volume for Postgres data
	    volumes: pg_data: # Explicitly sets the volume name
```
