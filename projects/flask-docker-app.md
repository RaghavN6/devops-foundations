# Flask Docker App

A simple Flask web application containerized with Docker to demonstrate the fundamentals of Docker images, containers, networking, and Python application deployment.

---

## 📌 Overview

This project demonstrates how to containerize a Python Flask application using Docker. It covers the complete workflow from writing the application and Dockerfile to building an image, running a container, and accessing the application through a web browser.

---

## ✨ Features

- Simple Flask web application
- Dockerized using a custom Dockerfile
- Dependency management with `requirements.txt`
- Optimized Docker layer caching
- Port mapping between host and container
- Accessible through a web browser

---

## 🛠 Technologies Used

- Python 3.12
- Flask
- Docker

---

## 📂 Project Structure

```
flask-docker-app/
│── app.py
│── Dockerfile
│── requirements.txt
│── .dockerignore
│── .gitignore
└── README.md
```

---

## 🚀 Usage

Build the Docker image:

```bash
docker build -t flask-app .
```

Run the container:

```bash
docker run -p 5000:5000 flask-app
```

Open your browser:

```
http://localhost:5000
```

---

## 📄 Example Output

```
Hello DevOps!
```

---

## 📚 Concepts Practiced

- Flask Basics
- HTTP Routing
- Docker Images
- Docker Containers
- Dockerfile Instructions
- Build Context
- Docker Layer Caching
- Container Networking
- Port Mapping
- Image Build Process

---

## 🧠 Docker Concepts Used

| Concept | Purpose |
|----------|---------|
| `FROM` | Select base image |
| `WORKDIR` | Set working directory |
| `COPY` | Copy files into image |
| `RUN` | Execute commands during build |
| `ENTRYPOINT` | Define container startup command |
| `EXPOSE` | Document application port |
| `docker build` | Build Docker image |
| `docker run` | Create and start container |
| `-p` | Publish container ports |

---

## 🎯 Learning Outcomes

Through this project I learned:

- How Docker images are built from a Dockerfile.
- The difference between images and containers.
- How to package a Python Flask application into a Docker image.
- How Docker layer caching speeds up rebuilds.
- The difference between `EXPOSE` and `-p`.
- Why applications inside containers should bind to `0.0.0.0`.
- How to build, run, and verify a containerized application.

---

## 🔮 Future Improvements

- Add Docker Compose support
- Integrate a PostgreSQL database
- Add Nginx as a reverse proxy
- Implement environment variables
- Add health checks
- Deploy to a cloud platform

---

## 👨‍💻 Author

**Raghav N**

GitHub: https://github.com/RaghavN6666

---

This project is part of my DevOps learning journey, focusing on Docker, containerization, and application deployment.