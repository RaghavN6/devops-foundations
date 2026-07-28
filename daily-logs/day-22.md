# 📅 Daily Learning Log

**Date:** July 29, 2026

## 🎯 Objective
Learn Docker Compose and understand how to manage multi-container applications using a single configuration file.

---

## 📚 Topics Covered
- Introduction to Docker Compose
- `compose.yml`
- Services
- `build` vs `image`
- `docker compose up`
- `docker compose up -d`
- `docker compose stop`
- `docker compose start`
- `docker compose down`
- Environment Variables
- Volumes in Docker Compose
- `depends_on`

---

## 🧠 Concepts Learned

### 1. Introduction to Docker Compose
- Docker Compose simplifies managing multi-container applications.
- Instead of running multiple `docker run` commands, the entire application is defined in a single YAML file.
- Applications can be started using one command:

```bash
docker compose up
```

---

### 2. `compose.yml`
- `compose.yml` is the default configuration file used by Docker Compose.
- It defines the application's services, networks, volumes, ports, and environment variables.
- Docker automatically looks for this file when running `docker compose up`.

---

### 3. Services
- Every container in a Compose project is defined as a **service**.
- The service name is also used for Docker's internal DNS.

Example:

```yaml
services:
  flask:
    image: flask-app
```

---

### 4. `build` vs `image`

**`build`**
- Used when building an image from a local `Dockerfile`.

```yaml
build: .
```

**`image`**
- Used when pulling an existing image from Docker Hub or another registry.

```yaml
image: mysql
```

Rule:
- Use **build** for applications you develop.
- Use **image** for official or pre-built images.

---

### 5. Starting and Stopping Applications

Start the application:

```bash
docker compose up
```

Run in the background:

```bash
docker compose up -d
```

View running services:

```bash
docker compose ps
```

Temporarily stop containers:

```bash
docker compose stop
```

Restart stopped containers:

```bash
docker compose start
```

Stop and remove containers:

```bash
docker compose down
```

---

### 6. Environment Variables
- Environment variables provide configuration without modifying application code.
- They are commonly used for database credentials, API keys, and application settings.

Example:

```yaml
environment:
  MYSQL_ROOT_PASSWORD: password
  MYSQL_DATABASE: mydb
```

---

### 7. Volumes in Docker Compose
- Volumes provide persistent storage for containers.
- The volume is mounted inside the service and declared at the bottom of the Compose file.

Example:

```yaml
services:
  mysql:
    image: mysql
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

- The service section **mounts** the volume.
- The bottom section **declares** the volume so Docker Compose can create and manage it.

---

### 8. `depends_on`
- Defines service startup order.
- Ensures dependent services start before the current service.

Example:

```yaml
depends_on:
  - mysql
```

- Guarantees startup order only.
- Does **not** guarantee that the dependent service is fully ready to accept connections.

---

## ❌ Mistakes Made Today
- Initially thought `build` and `image` should always be used together.
- Clarified that:
  - `build` is used for locally developed applications.
  - `image` is used for existing images from registries like Docker Hub.
- Refined the understanding that `depends_on` controls **startup order**, not service readiness.

---

## 🔑 Key Takeaways
- Docker Compose manages an entire application using a single configuration file.
- `compose.yml` is the default Compose configuration file.
- Services define the containers in the application.
- Use `build` for local applications and `image` for existing images.
- `docker compose up -d` is commonly used to run applications in the background.
- Environment variables separate configuration from application code.
- Volumes provide persistent storage for containers.
- `depends_on` defines startup order but does not wait for a service to become fully ready.

---

## 💭 Reflection
Today I learned how Docker Compose simplifies running multi-container applications. I also understand how services, build, image, environment variables, volumes, and `depends_on` work together in a Compose project.

---

## 📈 Progress Tracker
- ✅ Linux Fundamentals
- ✅ Git & GitHub
- ✅ Docker Basics
- ✅ Docker Images
- ✅ Docker Containers
- ✅ Dockerfile
- ✅ Container Management
- ✅ Docker Volumes & Bind Mounts
- ✅ Docker Networking
- ✅ Docker Compose
- 🔜 Docker Compose Project (Flask + MySQL)
- 🔜 Docker Best Practices
- 🔜 Docker Troubleshooting Commands
- 🔜 CI/CD with GitHub Actions