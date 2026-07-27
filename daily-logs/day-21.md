# 📅 Daily Learning Log

**Date:** July 28, 2026

## 🎯 Objective
Complete Docker Networking and understand how containers communicate with each other, the host machine, and external users.

---

## 📚 Topics Covered
- Docker Network Types
  - Bridge Network
  - Host Network
  - None Network
- Custom Bridge Networks
- Docker DNS
- Port Mapping (`-p`)

---

## 🧠 Concepts Learned

### 1. Docker Network Types
- **Bridge** is Docker's default network where containers receive their own private IP addresses.
- **Host** allows containers to share the host machine's network stack and IP address.
- **None** disables networking completely for the container.

### 2. Bridge Network
- Each container gets its own Docker-managed IP address.
- Containers on the same bridge network can communicate with one another.
- Best suited for multi-container applications like Flask, MySQL, and Redis.

### 3. Host Network
- Containers use the host machine's network directly.
- Containers do **not** receive separate Docker IP addresses.
- Sharing the host network can cause port conflicts if multiple containers use the same port.
- Mainly used for specialized networking or high-performance scenarios.

### 4. Custom Bridge Networks
Created a custom network:

```bash
docker network create my-network
```

Listed available networks:

```bash
docker network ls
```

Inspected network details:

```bash
docker network inspect my-network
```

Attached containers to the custom network:

```bash
docker run --network my-network
```

### 5. Docker DNS
- Docker automatically resolves container names to their IP addresses.
- Containers should communicate using container names instead of hardcoded IP addresses.
- This ensures applications continue working even if container IPs change.

Example:

```python
DB_HOST = "mysql"
```

instead of

```python
DB_HOST = "172.18.0.5"
```

### 6. Port Mapping (`-p`)
Learned how to expose applications running inside containers.

Syntax:

```bash
docker run -p HOST_PORT:CONTAINER_PORT image
```

Example:

```bash
docker run -p 8080:5000 flask-app
```

Meaning:
- Host port **8080** forwards traffic to container port **5000**.
- The application is accessed through:

```
http://localhost:8080
```

---

## ❌ Mistakes Made Today
- Initially thought the **Host Network** was a shared communication channel between containers.
- Confused the **host machine** with a **container**.
- Clarified that containers using the host network share the **host's network stack and IP**, while bridge networking provides each container with its own IP address.

---

## 🔑 Key Takeaways
- Bridge is the default and recommended network for most Docker applications.
- Host networking shares the host machine's network instead of creating a separate Docker network.
- Docker DNS allows containers to communicate using container names instead of IP addresses.
- Port mapping connects a host port to a container port.
- The left side of `HOST_PORT:CONTAINER_PORT` is the port users access from the host.

---

## 💭 Reflection
Today I completed Docker Networking and gained a solid understanding of how containers communicate using Docker networks, DNS, and port mapping. I also clarified the difference between bridge and host networking.

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
- 🔜 Docker Compose