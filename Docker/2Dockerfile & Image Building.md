### Docker Module 2 — Dockerfile & Image Building

Beginner → Junior DevOps practical level

In Module 1, you downloaded and ran an existing image (`nginx:alpine`). Now you'll learn to create your own Docker image for an application.

### Module 2 roadmap

### 1

Dockerfile fundamentals

What it is and how a build works

### 2

Essential instructions

FROM, WORKDIR, COPY, RUN

### 3

Starting the application

CMD vs ENTRYPOINT

### 4

Configuration and ports

ENV, ARG, EXPOSE

### 5

Build your own image

docker build, tags and context

### 6

Image layers and cache

Why build order matters

### 7

Clean builds

.dockerignore and multi-stage basics

### 8

Hands-on project

Build and run your own web app

### 9

Troubleshooting

Fix common build and startup errors

### Lesson 1 — What is a Dockerfile?

A Dockerfile is a text file containing instructions Docker uses to build an image.

For example, suppose you have a Python application. You need to tell Docker:

* Which base environment to use.

* Where to put the application files.

* Which dependencies to install.

* Which command starts the application.

A Dockerfile records those instructions.

### Visual workflow

Dockerfile

Build instructions

docker build

Reads instructions and creates the image

Image

Reusable application package

docker run

Creates and starts a container

### Lesson 2 — Understand the essential instructions

Here is a basic Dockerfile:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

Don't worry if it looks complicated. Let's understand each instruction.

| Instruction | Meaning                                          | Simple example           |
| ----------- | ------------------------------------------------ | ------------------------ |
| FROM        | Selects the base image                           | FROM python:3.12-slim    |
| WORKDIR     | Sets the working directory                       | WORKDIR /app             |
| COPY        | Copies files into the image                      | COPY app.py .            |
| RUN         | Executes a command during image building         | RUN pip install ...      |
| CMD         | Sets the default command when a container starts | CMD ["python", "app.py"] |

Important distinction:

* `RUN` executes while building the image.

* `CMD` executes when the container starts.

For example, installing a Python library is a build-time operation. Starting your Python application is a run-time operation.

### Lesson 3 — Your first Dockerfile project

We'll build a tiny Python web application and run it in Docker.

### Step 1: Create a project directory

In Ubuntu/WSL, run:

```bash
mkdir docker-module2
cd docker-module2
```

### Step 2: Create the application

Run:

```bash
nano app.py
```

Paste this code:

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
import os

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header("Content-type", "text/plain")
        self.end_headers()
        self.wfile.write(b"My first Docker application works!")

port = int(os.environ.get("PORT", "8000"))

server = HTTPServer(("0.0.0.0", port), Handler)
print(f"Server running on port {port}")
server.serve_forever()
```

Save in Nano with `Ctrl+O`, press `Enter`, then `Ctrl+X`.

This application listens on port `8000`. The address `0.0.0.0` allows it to accept connections through the container's network interface.

### Step 3: Create the Dockerfile

```bash
nano Dockerfile
```

Paste:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

EXPOSE 8000

CMD ["python", "app.py"]
```

Save and exit.

Your folder should look like this:

docker-module2/

`Dockerfile`

`app.py`

### Step 4: Build your image

Run this command from inside the `docker-module2` directory:

```bash
docker build -t my-python-app:1.0 .
```

Understand the command:

* `docker build`: build an image.

* `-t`: assign a name and tag.

* `my-python-app:1.0`: image name and version.

* `.`: use the current directory as the build context.

The build context is the set of files made available to Docker during the build. The final dot is important.

### Step 5: Verify the image

```bash
docker images
```

You should find `my-python-app` with tag `1.0`.

### Step 6: Run your application

```bash
docker run -d --name my-python-container -p 8000:8000 my-python-app:1.0
```

Open:

`http://localhost:8000`

Expected response:

```text
My first Docker application works!
```

### Step 7: Check logs

```bash
docker logs my-python-container
```

Check the running container:

```bash
docker ps
```

### Step 8: Clean up

```bash
docker stop my-python-container
docker rm my-python-container
```

The image remains available for later use.

### Lesson 4 — More Dockerfile instructions

These are the next instructions you should recognise and practise.

| Instruction | Purpose                                             | Key point                                     |
| ----------- | --------------------------------------------------- | --------------------------------------------- |
| ENV         | Sets environment variables in the image/container   | Values can persist in image configuration     |
| ARG         | Defines a build-time variable                       | Not suitable for secrets                      |
| EXPOSE      | Documents the intended listening port               | Does not publish the port                     |
| USER        | Selects the user for later instructions and runtime | Prefer non-root for application processes     |
| ENTRYPOINT  | Defines the main executable                         | Often used with CMD for default arguments     |
| ADD         | Copies files with additional features               | Prefer COPY unless those features are needed  |
| VOLUME      | Declares a mount point                              | Does not itself provide a backup              |
| HEALTHCHECK | Configures a health-check command                   | Health status is separate from process status |

### `CMD` vs `ENTRYPOINT`

This is a common interview question.

| CMD                                             | ENTRYPOINT                            |
| ----------------------------------------------- | ------------------------------------- |
| Provides a default command or arguments         | Defines the main executable           |
| Often easy to override when running a container | Normally stays as the main executable |
| Can supply default arguments to ENTRYPOINT      | Commonly paired with CMD              |

Example:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

The default starts `python app.py`. Supplying a different command argument at runtime can replace the default `CMD` arguments.

Use JSON/exec form for `CMD` and `ENTRYPOINT` in most application images so the application receives signals correctly.

### Lesson 5 — Image layers and build cache

Docker builds images in steps. Many instructions create or contribute to filesystem layers.

For example:

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
```

Docker can reuse cached results when the relevant inputs and instructions haven't changed.

Why order matters: if you copy all source code before installing dependencies, changing a single source file may invalidate the dependency-installation cache. Copying dependency files first often makes rebuilds faster.

### Lesson 6 — `.dockerignore`

Create a file named `.dockerignore` in the project directory:

```text
.git
.env
__pycache__
*.pyc
venv
```

This prevents matching files from being sent as part of the build context.

It helps reduce unnecessary build input and the risk of accidentally including local files. It is not a replacement for proper secret management.

### Lesson 7 — Multi-stage builds

Multi-stage builds use more than one `FROM` stage so build tools and intermediate files can be left out of the final runtime image.

Build stage

Compiler, development tools, dependencies

Runtime stage

Only required application output and runtime dependencies

Multi-stage builds are especially useful for compiled applications. For our small Python example, they aren't necessary yet.

### Module 2 mini-test

1. Which instruction selects the base image?

A. FROM

B. RUN

C. EXPOSE

2. When does \`RUN\` execute?

A. Only when a user opens the website

B. During image building

C. When the container is removed

3. What does the final \`.\` mean in \`docker build -t app:1.0 .\`?

A. Delete the current directory

B. Use the current directory as build context

C. Publish port 80

4. Does \`EXPOSE 8000\` automatically publish port 8000 on your computer?

A. Yes

B. No

5. What is the main benefit of \`.dockerignore\`?

A. Exclude unnecessary files from the build context

B. Automatically scan every image for vulnerabilities

C. Start containers automatically

6. What is the purpose of a multi-stage build?

A. Run every container as root

B. Separate build steps from the final runtime image

C. Replace Docker Compose

Submit test

### Module 2 completion checklist

* Created a Dockerfile.

* Built `my-python-app:1.0`.

* Ran the container and opened the web page.

* Checked logs and container status.

* Explained `FROM`, `WORKDIR`, `COPY`, `RUN`, `CMD`, and `EXPOSE`.

* Understood build cache and `.dockerignore`.

* Understood the purpose of multi-stage builds.

Do the hands-on project before moving forward. If a command fails, send me the exact command and error output, and we'll fix that specific issue.
