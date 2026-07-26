# 📅 DevOps Learning Log — Day 5
**Date:** 27 July 2026

---

# 🎯 Objective

Today's objective was to learn how to manage Docker containers after they are created and understand how Docker handles persistent data using Docker Volumes.

---

# 📚 Topics Covered

## Docker Container Management

- docker ps
- docker ps -a
- docker images
- docker stop
- docker start
- docker restart
- docker logs
- docker exec
- docker rm
- docker rmi

## Docker Volumes (Part 1)

- Ephemeral containers
- Persistent storage
- Creating Docker volumes
- Inspecting volumes
- Mounting volumes
- Sharing a volume between multiple containers (Concept)

---

# 🧠 Concepts Learned

## Images vs Containers

Reinforced the distinction between Docker images and containers.

- Image = Read-only blueprint/template
- Container = Running instance created from an image

One image can be used to create multiple independent containers.

---

## Viewing Docker Resources

Learned the difference between:

```bash
docker ps
```

Shows only running containers.

```bash
docker ps -a
```

Shows all containers including stopped and exited ones.

```bash
docker images
```

Shows all locally available Docker images.

One important realization was that **images don't have a running state**. Only containers can be running, stopped or exited.

---

## Container Lifecycle

Learned how Docker manages containers after creation.

```bash
docker run
```

Creates a **new** container from an image.

```bash
docker start
```

Starts an **existing stopped** container.

```bash
docker restart
```

Restarts a running container and can also start a stopped container.

---

## Docker Logs

Learned that:

```bash
docker logs <container>
```

Displays everything the application writes to:

- stdout
- stderr

Useful for identifying:

- Runtime errors
- Python exceptions
- Flask startup messages
- Application crashes

---

## Debugging Mindset

A very important discussion today was understanding that **logs are not enough**.

If the application is healthy but:

- Wrong port is mapped
- Wrong URL is used
- Networking issue exists

The logs may show no errors.

A proper debugging workflow should be:

```
Container Running?
        ↓
docker ps
        ↓
Application Healthy?
        ↓
docker logs
        ↓
Ports Correct?
        ↓
docker ps
        ↓
Inspect Container
        ↓
docker exec
```

This reinforced the idea of debugging systematically rather than relying on one command.

---

## Docker Exec

Learned:

```bash
docker exec
```

Runs commands inside a running container.

Examples:

```bash
docker exec flask1 ls
```

```bash
docker exec flask1 python --version
```

```bash
docker exec -it flask1 bash
```

Key distinction:

- `docker logs` answers **"What did my application print?"**
- `docker exec` answers **"What exists inside my container?"**

---

## Removing Containers & Images

Learned the difference between:

```bash
docker rm
```

Removes containers.

```bash
docker rmi
```

Removes images.

Also learned that:

- Running containers cannot be removed.
- Images cannot be removed while containers still depend on them.

Correct cleanup order:

```bash
docker stop
docker rm
docker rmi
```

---

## Docker Volumes

Containers are **ephemeral**, meaning data stored inside them is temporary.

Without volumes:

```
Container
│
├── users.txt
└── logs.txt

docker rm
      ↓
Everything deleted
```

Docker Volumes solve this problem.

A volume exists independently of containers.

Deleting the container does **not** delete the volume.

---

## Creating Volumes

Created a Docker volume using:

```bash
docker volume create project-data
```

Learned that this creates only storage.

No image.

No container.

Just persistent storage.

---

## Mounting Volumes

Example:

```bash
docker run -v project-data:/app flask-app
```

Anything written inside:

```
/app
```

is actually stored in the Docker volume.

Even if the container is deleted, the data remains.

---

## Shared Volumes

Learned that a single Docker volume can be mounted into multiple containers.

```
Container A
      │
      ▼
 Shared Volume
      ▲
      │
Container B
```

Both containers can access the same files.

Also discussed that applications writing simultaneously to the same files may create conflicts depending on the workload.

---

# ❌ Mistakes Made Today

## Mistake 1

Initially assumed:

```bash
docker run flask1
```

could restart an existing container.

### Correction

`docker run` always creates a **new container**.

To reuse an existing stopped container:

```bash
docker start flask1
```

---

## Mistake 2

Assumed:

```bash
docker restart
```

only works on running containers.

### Correction

Docker can also restart a stopped container by simply starting it.

However,

```bash
docker start
```

is the preferred command because it clearly communicates the intended action.

---

## Mistake 3

Initially thought `docker logs` could diagnose every Docker issue.

### Correction

Logs only show what happens **inside** the application.

Problems involving:

- Wrong ports
- Networking
- Browser URL
- Firewall

require additional investigation using other Docker commands.

---

# 💡 Key Takeaways

- Images are blueprints; containers are running instances.
- `docker run` creates new containers.
- `docker start` reuses stopped containers.
- `docker logs` helps debug application-level issues.
- `docker exec` allows inspecting the container directly.
- Containers are temporary (ephemeral).
- Docker Volumes provide persistent storage.
- Volumes exist independently of containers.
- One volume can be mounted into multiple containers.

---

# 📝 Reflection

Today's session significantly improved my understanding of how Docker works beyond simply building and running containers.

The biggest conceptual shift was learning to think in terms of **what question I'm trying to answer** before choosing a Docker command. Instead of treating `docker logs` as the solution to every issue, I now understand that different commands serve different debugging purposes.

I also learned one of Docker's core philosophies: **containers should be disposable, but data should not be**. Docker Volumes solve this by separating application storage from the container itself, allowing data to persist even when containers are recreated.

Overall, today's lesson strengthened both my Docker fundamentals and my approach to troubleshooting real-world containerized applications.