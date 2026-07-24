# Daily DevOps Log – Dockerizing a Flask Application

**Date:** 25/07/2026

---

## 📌 Objective

Learn how to containerize a Python Flask application using Docker while understanding every concept behind the process instead of simply following commands.

---

## 📚 Topics Covered

- Flask fundamentals
- HTTP requests and routes
- `requirements.txt`
- Dockerfile instructions
  - `FROM`
  - `WORKDIR`
  - `COPY`
  - `RUN`
  - `ENTRYPOINT`
  - `EXPOSE`
- Docker build context
- Docker layer caching
- Build vs Run lifecycle
- `EXPOSE` vs `-p`
- Container networking (`127.0.0.1` vs `0.0.0.0`)

---

## 🛠 What I Built

- Created a Flask application.
- Created `requirements.txt`.
- Wrote a Dockerfile from scratch.
- Built the Docker image.
- Ran the Docker container.
- Verified the application in the browser.
- Published the completed project to GitHub.

---

## ❌ Mistakes Made & Fixes

### 1. Forgot the project structure

**Mistake**

Initially wasn't sure what files were required for a Flask Docker project.

**Fix**

Learned the complete structure:

```
app.py
Dockerfile
requirements.txt
.dockerignore
```

**Reasoning**

Every file has a specific responsibility:
- `app.py` contains the application.
- `requirements.txt` manages dependencies.
- `Dockerfile` defines how to build the image.
- `.dockerignore` prevents unnecessary files from being copied.

---

### 2. Confused `pip install Flask` with `pip install -r requirements.txt`

**Mistake**

Thought `pip install -r Flask` was the correct command.

**Fix**

Learned:

```bash
pip install Flask
```

installs a single package, while

```bash
pip install -r requirements.txt
```

installs every dependency listed in the file.

**Reasoning**

Projects should store dependencies in a file so every developer installs the exact same packages.

---

### 3. Forgot Flask syntax

**Mistake**

Initially wrote incorrect imports and application initialization.

**Fix**

Correctly wrote:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello DevOps!"
```

**Reasoning**

Understanding the purpose of each line was more valuable than memorizing syntax.

---

### 4. Forgot why `requirements.txt` is copied before the application

**Mistake**

Initially only knew the order without fully understanding it.

**Fix**

Learned about Docker layer caching.

**Reasoning**

Dependencies change less frequently than source code.

Copying `requirements.txt` first allows Docker to reuse cached dependency layers instead of reinstalling packages every build.

---

### 5. Tried to start Flask using `RUN`

**Mistake**

Thought:

```dockerfile
RUN python app.py
```

would start the application.

**Fix**

Used:

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

**Reasoning**

`RUN` executes during image build.

`ENTRYPOINT` executes when the container starts.

---

### 6. Mixed up `CMD` and `ENTRYPOINT`

**Mistake**

Thought CMD referred to process completion time.

**Fix**

Learned:

- `ENTRYPOINT` defines the main executable.
- `CMD` provides default arguments or a default command that can be overridden.

**Reasoning**

Both execute at container runtime but serve different purposes.

---

### 7. Mixed up `EXPOSE` and `-p`

**Mistake**

Thought:

```
EXPOSE -p 5000:5000
```

was valid.

**Fix**

Learned:

Dockerfile:

```dockerfile
EXPOSE 5000
```

Runtime:

```bash
docker run -p 5000:5000 flask-app
```

**Reasoning**

`EXPOSE` documents the intended port.

`-p` publishes the container port to the host.

---

### 8. Mixed up `docker build` and `docker run`

**Mistake**

Attempted:

```bash
docker build -t -p 5000:5000 flask-app .
```

**Fix**

Separated responsibilities:

```bash
docker build -t flask-app .
```

then

```bash
docker run -p 5000:5000 flask-app
```

**Reasoning**

Building creates an image.

Running creates a container from that image.

---

### 9. Confused build context with COPY destination

**Mistake**

Thought the `.` in

```bash
docker build -t flask-app .
```

meant destination.

**Fix**

Learned:

- In `docker build`, `.` is the build context.
- In `COPY`, `.` is the destination inside the image.

**Reasoning**

The same symbol has different meanings depending on the command.

---

### 10. Didn't understand why Flask wouldn't work inside Docker

**Mistake**

Didn't know why everyone changes:

```python
app.run()
```

to

```python
app.run(host="0.0.0.0", port=5000)
```

**Fix**

Learned container networking.

**Reasoning**

`127.0.0.1` only accepts connections from inside the container.

`0.0.0.0` accepts connections from every network interface, allowing Docker's port mapping to work.

---

### 11. Tried opening the application from the terminal

**Mistake**

Typed:

```
http://127.0.0.1
```

inside the terminal.

**Fix**

Opened the URL in a web browser instead.

**Reasoning**

The terminal starts the server.

The browser sends HTTP requests to the server.

---

## ✅ Final Outcome

Successfully built and containerized my first Flask application using Docker.

Verified the application through:

```
http://localhost:5000
```

Successfully pushed the completed project to GitHub.

---

## 💭 Reflection

Today was one of the longest yet smoothest projects I've completed. Instead of focusing on memorizing commands, I spent time understanding why each Docker instruction exists and how Flask interacts with Docker. Whenever I made a mistake, it was usually because I mixed up similar concepts rather than not knowing them. By the end of the project, I was able to build, run, and verify the application without any build or runtime errors. This project gave me a much clearer understanding of Docker's workflow from source code to a running container and increased my confidence for learning Docker Compose next.