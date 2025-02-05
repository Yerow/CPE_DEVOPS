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

| Command | Description |
| --- | --- |
| `docker-compose up -d` | Starts all services in the background. |
| `docker-compose down` | Stops and removes all services, networks, and volumes. |
| `docker-compose build` | Builds/Rebuilds the services from `docker-compose.yml`. |
| `docker-compose ps` | Lists running services. |
| `docker-compose logs -f` | Streams live logs of all services (useful for debugging). |
| `docker-compose exec <service> bash` | Opens a terminal inside a running container. |
| `docker-compose restart <service>` | Restarts a specific service. |
| `docker-compose rm -f` | Removes stopped containers. |
| `docker-compose stop` | Stops all running containers without removing them. |
| `docker-compose up --scale backend=3` | Runs 3 instances of the `backend` service. |

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
```

---

# GIT & CI/CD

### ❓ 2-1 What are testcontainers?

**Testcontainers** is a **Java library** that provides lightweight, **throwaway instances of databases, message brokers, and other services** in Docker containers. It is primarily used for **integration testing**, allowing developers to test their applications in an environment that closely resembles production.

---

#### **🛠 Key Features of Testcontainers**

1. **Runs Dependencies in Docker** 🐳
    
    - Easily starts and stops Docker containers for databases (PostgreSQL, MySQL, MongoDB, etc.), message brokers (Kafka, RabbitMQ), and other services.
2. **Automatic Cleanup** 🗑️
    
    - Containers are automatically destroyed after the test finishes, preventing environment pollution.
3. **Simplifies Integration Testing** 🔄
    
    - Eliminates the need for local database installations, making tests portable and reproducible.
4. **Works with Popular Testing Frameworks** ✅
    
    - Compatible with JUnit, TestNG, Spring Boot, etc.
5. **Supports Custom Containers** 🔧
    
    - You can define your own containers for specific dependencies.

---

### 🔍 2-2 Document your Github Actions configurations.

#### Github action config
`main.yml` 
```yml
name: CI devops 2025
on:
  #to begin you want to launch this job in main and develop
  push:
    branches:
      - main
      - develop 
  pull_request:

jobs:
  test-backend: 
    runs-on: ubuntu-22.04
    steps:
     #checkout your github code using actions/checkout@v2.5.0
      - uses: actions/checkout@v4

     #do the same with another action (actions/setup-java@v3) that enable to setup jdk 21
      - name: Set up JDK 21
        uses: actions/setup-java@v3
        with:
          distribution: 'corretto'
          java-version: '21'

     #finally build your app with the latest command
      - name: Build and test with Maven
        run: mvn clean verify
        working-directory: simple-api
```

---

### ❓ 2-3 For what purpose do we need to push docker images?

We push Docker images to a registry so they can be **shared, deployed, and scaled** across different environments.

---

### ❓ 2-4 Document your quality gate configuration.

Quality Gate Config
```yml
sonarcloud-github-action:
    needs: test-backend  # Exécute Sonar uniquement si les tests passent
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: 21
          distribution: 'corretto'

      - name: Cache SonarQube packages
        uses: actions/cache@v4
        with:
          path: ~/.sonar/cache
          key: ${{ runner.os }}-sonar
          restore-keys: ${{ runner.os }}-sonar

      - name: Cache Maven packages
        uses: actions/cache@v4
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-m2

      - name: Build and analyze
        working-directory: simple-api  
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=Yerow_CPE_DOCKER
```

ajout de ces deux lignes dans le pom.xml de simple_api
```xml
<sonar.organization>yerow</sonar.organization>
<sonar.host.url>https://sonarcloud.io</sonar.host.url>
```

---

# Ansible

### 📄 3-1 Document your inventory and base commands

Testing my inventory with ping command
```bash
ansible all -i inventories/setup.yml -m ping
```

Request server to get OS distribution
```bash
ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
```

Remove apache2 from my machine
```bash
ansible all -i inventories/setup.yml -m apt -a "name=apache2 state=absent" --become
```

Here is my `setup.yml` in my inventory
```yml
all:
 vars:
	xansible_user: admin
   ansible_ssh_private_key_file: ~/.ssh/id_rsa
 children:
   prod:
     hosts: mohamed.gueffaf.takima.cloud
```

Executing my playbook
```bash
ansible-playbook -i inventories/setup.yml playbook.yml
```
You can check your playbooks before playing them using the option: `--syntax-check`

Installing docker

🔹 Ansible Command to Check Docker Status using `systemctl`
```bash
ansible all -i inventories/setup.yml -m command -a "systemctl is-active docker" -b
```
✅ Expected Output (if Docker is running)
```bash
active
```

Creating a docker role and moving the installation task there :
```bash
ansible-galaxy init roles/docker
```

---

### 📄 3-2 Document your playbook

`install-docker.yml`
```yml
- hosts: all

  gather_facts: true

  become: true

  roles:

    - docker
```

`roles/docker/tasks/main.yml`
```yml
# tasks file for roles/docker
- name: Install required packages
  apt:
    name:
      - apt-transport-https
      - ca-certificates
      - curl
      - gnupg
      - lsb-release
      - python3-venv
    state: latest
    update_cache: yes

# Add Docker’s official GPG key
- name: Add Docker GPG key
  apt_key:
    url: https://download.docker.com/linux/debian/gpg
    state: present

# Set up the Docker stable repository
- name: Add Docker APT repository
  apt_repository:
    repo: "deb [arch=amd64] https://download.docker.com/linux/debian {{ ansible_facts['distribution_release'] }} stable"
    state: present
    update_cache: yes

# Install Docker (FIXED INDENTATION)
- name: Install Docker
  apt:
    name: docker-ce
    state: present

# Install Python3 and pip3
- name: Install Python3 and pip3
  apt:
    name:
      - python3
      - python3-pip
    state: present

# Create a virtual environment for Python packages
- name: Create a virtual environment for Docker SDK
  command: python3 -m venv /opt/docker_venv
  args:
    creates: /opt/docker_venv  # Only runs if this directory doesn’t exist

# Install Docker SDK for Python in the virtual environment
- name: Install Docker SDK for Python in virtual environment
  command: /opt/docker_venv/bin/pip install docker

# Ensure Docker is running
- name: Make sure Docker is running
  service:
    name: docker
    state: started
  tags: docker
```

---

### 📄 3-3 Document your docker_container tasks configuration.

#### ansible/
`deploy.yml`
```yml
- hosts: all
  gather_facts: true
  become: true

  roles:
    - docker
    - network
    - env-setup
    - database
    - app
    - proxy
```

##### roles/
`app/tasks/main.yml`
```yml
- name: Run Simple API
  community.docker.docker_container:
    name: simple-api
    image: mohaguef/tp-devops-simple-api:latest
    pull: yes
    env_file: /home/admin/.env  # Use the copied .env file
    networks:
      - name: my-network
    restart_policy: unless-stopped
```

`database/tasks/main.yml`
```yml
- name: Run PostgreSQL database
  community.docker.docker_container:
    name: database
    image: mohaguef/tp-devops-database:latest
    pull: yes
    env_file: /home/admin/.env  # Use the copied .env file
    volumes:
      - db-volume:/var/lib/postgresql/data
    networks:
      - name: my-network
    restart_policy: unless-stopped
```

`network/tasks/main.yml`
```yml
- name: Create Docker network
  community.docker.docker_network:
    name: my-network
    driver: bridge
```

`proxy/tasks/main.yml`
```yml
- name: Run Reverse Proxy
  community.docker.docker_container:
    name: reverse_proxy
    image: mohaguef/tp-devops-http-server:latest
    pull: yes
    ports:
      - "80:80"
    networks:
      - name: my-network
    restart_policy: unless-stopped
```
---