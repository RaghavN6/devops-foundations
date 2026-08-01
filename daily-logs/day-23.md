# 📅 Daily Learning Log

**Date:** August 2, 2026

## 🎯 Objective

Build a multi-container application using Docker Compose and understand how application containers communicate with database containers while handling startup dependencies and connection failures.

---

## 📚 Topics Covered

- Docker Compose Project (Flask + MySQL)
- Multi-Container Applications
- Flask ↔ MySQL Communication
- Internal Docker DNS
- Environment Variables
- Database Connection Configuration
- Retry Logic
- Transient vs Permanent Failures
- Exception Handling
- HTTP 503 Service Unavailable

---

## 🧠 Concepts Learned

### 1. Multi-Container Architecture

Built a Docker Compose application consisting of:

- Flask Application
- MySQL Database

Instead of running containers individually, both services are managed through a single `compose.yml` file.

Application architecture:

```text
Browser
    │
    ▼
 Flask Container
    │
Docker DNS
    │
    ▼
 MySQL Container
```

---

### 2. Docker DNS

Learned that containers communicate using service names rather than IP addresses.

Example:

```python
host = "mysql"
```

Reason:

Docker automatically provides an internal DNS server that resolves the service name `mysql` to the database container's current IP address.

Benefits:

- No hardcoded IP addresses.
- Containers continue communicating even if the database container receives a different IP after being recreated.
- Easier maintenance of multi-container applications.

---

### 3. Why the Database Host is `"mysql"`

Initially explored why the Flask application connects to:

```python
host = "mysql"
```

instead of:

```python
host = "localhost"
```

Understanding:

The Flask container and MySQL container are separate containers.

`localhost` inside the Flask container refers only to the Flask container itself.

Using the MySQL service name allows Docker's internal DNS to route requests to the database container.

---

### 4. Startup Order vs Readiness

Learned the purpose of:

```yaml
depends_on:
  - mysql
```

Understanding:

`depends_on` only ensures that Docker starts the MySQL container before the Flask container.

It does **not** guarantee that MySQL has finished initializing and is ready to accept connections.

This means the Flask application may attempt to connect before the database is available.

---

### 5. Retry Logic

Discussed why retry mechanisms are commonly used when connecting to databases.

Transient failures include:

- Database still starting
- Temporary network delays
- Container initialization

In these cases, retrying the connection is appropriate.

However, retrying forever is not always desirable.

---

### 6. Transient vs Permanent Failures

A key realization today was distinguishing between different types of failures.

#### Transient Failure

Examples:

- Database still starting
- Temporary network issue

Appropriate response:

- Retry connection.

#### Permanent Failure

Examples:

- Incorrect username
- Incorrect password
- Invalid database name

Appropriate response:

- Stop retrying.
- Report the error.

This distinction prevents applications from endlessly retrying problems that cannot resolve themselves.

---

### 7. Exception Handling

Discussed using Python exception handling to distinguish different database errors.

General workflow:

```text
Attempt Connection
        │
        ▼
Connection Successful?
        │
 ┌──────┴──────┐
 │             │
Yes            No
 │             │
 ▼             ▼
Continue   Identify Error
                │
      ┌─────────┴─────────┐
      │                   │
 Temporary           Permanent
 Failure              Failure
      │                   │
Retry Connection     Return Error
```

This allows the application to respond differently depending on the reason the connection failed.

---

### 8. HTTP 503 Service Unavailable

Discussed appropriate responses when the database is temporarily unavailable.

Instead of crashing or returning misleading responses, the application should return:

```text
HTTP 503 Service Unavailable
```

Reason:

The application itself is functioning, but one of its required dependencies is temporarily unavailable.

This provides useful information during debugging and accurately reflects the system's state.

---

## 🛠️ Docker Compose Configuration

### Dockerfile

Created a Dockerfile that:

- Uses Python 3.12
- Sets the working directory
- Copies dependency definitions
- Installs required packages
- Copies the application
- Starts Flask using `CMD`

---

### Compose File

Configured two services:

- Flask
- MySQL

Also configured:

- Persistent MySQL volume
- Environment variables
- Local image build for Flask

---

## ❌ Mistakes Made Today

### Mistake 1

Initially assumed:

```python
host = "localhost"
```

would connect to the database.

### Correction

Each container has its own network namespace.

The Flask container must connect using:

```python
host = "mysql"
```

which Docker resolves through its internal DNS.

---

### Mistake 2

Initially thought:

```yaml
depends_on
```

waited until MySQL was fully operational.

### Correction

`depends_on` controls startup order only.

Applications should still handle connection failures gracefully.

---

### Mistake 3

Initially considered retrying every failed database connection indefinitely.

### Correction

Only temporary failures should be retried.

Permanent failures such as invalid credentials should fail immediately.

---

### Mistake 4

Initially viewed every connection failure as identical.

### Correction

Applications should distinguish between:

- Connection failures
- Authentication failures

Different problems require different responses.

---

## 🔑 Key Takeaways

- Docker Compose simplifies multi-container applications.
- Containers communicate using Docker's internal DNS.
- Service names should be used instead of hardcoded IP addresses.
- `depends_on` controls startup order but not service readiness.
- Retry logic is appropriate only for temporary failures.
- Authentication failures should not be retried indefinitely.
- Exception handling allows applications to respond appropriately to different failure types.
- HTTP 503 accurately indicates that a required dependency is temporarily unavailable.

---

## 💭 Reflection

Today's session shifted my focus from simply running multiple containers to thinking about how distributed applications behave when services depend on one another.

The most valuable realization was that not every failure should be treated the same. Instead of assuming every connection problem requires another retry, I learned to distinguish between temporary failures, which may resolve over time, and permanent failures, which require immediate attention.

I also developed a much clearer understanding of Docker's internal networking. Using service names instead of IP addresses, understanding the limitations of `depends_on`, and recognizing the importance of graceful error handling made the project feel much closer to how real-world containerized applications are designed.

Rather than memorizing Docker Compose syntax, today's session emphasized reasoning about system behavior, reliability, and maintainability—an approach that aligns well with building production-ready applications.

---

## 📈 Progress Tracker

- ✅ Linux Fundamentals
- ✅ Git & GitHub
- ✅ Bash Scripting & Automation
- ✅ Docker Basics
- ✅ Docker Images
- ✅ Docker Containers
- ✅ Dockerfile
- ✅ Container Management
- ✅ Docker Volumes & Bind Mounts
- ✅ Docker Networking
- ✅ Docker Compose
- ✅ Docker Compose Project (Flask + MySQL)
- 🔜 Docker Best Practices
- 🔜 Docker Troubleshooting
- 🔜 CI/CD with GitHub Actions