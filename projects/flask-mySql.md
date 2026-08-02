# 🐳 Flask + MySQL with Docker Compose

A Docker Compose-based multi-container application demonstrating how a Python Flask application communicates with a MySQL database using Docker's internal networking, persistent volumes, and environment-based configuration.

---

# 📌 Overview

This project demonstrates how Docker Compose simplifies the deployment and management of multi-container applications.

The application consists of two services:

- **Flask** – Web application
- **MySQL** – Relational database

Docker Compose orchestrates both containers, allowing them to communicate using Docker's built-in DNS while maintaining persistent database storage through Docker Volumes.

---

# 🏗️ Architecture

```text
                 HTTP Request
                       │
                       ▼
              +----------------+
              | Flask Container|
              +----------------+
                       │
         Docker Internal DNS
        (Host = "mysql")
                       │
                       ▼
             +----------------+
             | MySQL Container|
             +----------------+
                       │
                       ▼
               Docker Volume
              (Persistent Data)
```

---

# ✨ Features

- Multi-container application using Docker Compose
- Flask web application
- MySQL database container
- Internal Docker DNS for service discovery
- Persistent database storage using Docker Volumes
- Environment variable configuration
- Automatic image building using Docker Compose
- Clean separation between application and database services

---

# 🛠️ Technologies Used

- Docker
- Docker Compose
- Python 3.12
- Flask
- MySQL
- MySQL Connector/Python

---

# 📂 Project Structure

```text
flask-mysql-compose-app/
│
├── app.py
├── Dockerfile
├── compose.yml
├── requirements.txt
└── README.md
```

---

# ⚙️ Prerequisites

- Docker
- Docker Compose

Verify installation:

```bash
docker --version
docker compose version
```

---

# 🚀 Getting Started

## Clone the Repository

```bash
git clone git@github.com:RaghavN6666/flask-mySql.git
```

Navigate into the project:

```bash
cd flask-mySql
```

---

## Build and Start the Application

```bash
docker compose up --build
```

Docker Compose will:

- Build the Flask image
- Pull the MySQL image
- Create a Docker network
- Create a Docker volume
- Start both containers

---

## Access the Application

Open:

```
http://localhost:5000
```

Expected response:

```
Hello DevOps!
```

---

# 📦 Services

## Flask

Responsibilities:

- Handles HTTP requests
- Connects to MySQL
- Runs inside its own container

Built from the local Dockerfile.

---

## MySQL

Responsibilities:

- Stores application data
- Runs as a separate container
- Uses a Docker Volume for persistent storage

Pulled from the official MySQL image.

---

# 🌐 Docker Networking

Docker Compose automatically creates an isolated network for the project.

Instead of connecting using IP addresses, the Flask application communicates with MySQL using the service name:

```python
host = "mysql"
```

Docker's internal DNS resolves:

```text
mysql
        ↓
Container IP Address
```

This allows containers to communicate even if IP addresses change.

---

# 💾 Persistent Storage

The MySQL service stores its data inside a Docker Volume.

Benefits:

- Database survives container recreation
- Data is separated from the container lifecycle
- Easier backups and maintenance

---

# 📚 Concepts Demonstrated

This project demonstrates practical understanding of:

- Docker Compose
- Multi-container architecture
- Docker Networking
- Docker DNS
- Docker Volumes
- Environment Variables
- Docker Images
- Dockerfiles
- Container Communication
- Flask
- MySQL Integration

---

# 🎯 Learning Outcomes

Through this project I learned:

- How Docker Compose manages multiple containers.
- How containers communicate using Docker DNS.
- Why service names are used instead of IP addresses.
- How Docker Volumes provide persistent storage.
- The difference between startup order and service readiness.
- How to configure containerized applications using environment variables.

---

# 🔮 Future Improvements

- Add MySQL health checks
- Add automatic retry logic for database connections
- Use a `.env` file for configuration
- Add Docker Compose profiles
- Add Nginx as a reverse proxy
- Integrate with GitHub Actions
- Deploy to AWS EC2

---

# 👨‍💻 Author

**Raghav N**

GitHub: https://github.com/RaghavN6

---

# ⭐ Project Status

**Completed — Docker Compose Project as part of my DevOps Learning Roadmap**