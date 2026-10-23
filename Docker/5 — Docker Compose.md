### Docker Module 5 — Docker Compose

Beginner → Junior DevOps | Simple English + Hinglish | Hands-on project

In the previous modules, you learned Docker containers, Dockerfiles, networking, and volumes. Now let's combine them.

Problem: Suppose your application needs three containers:

* Frontend — website UI

* Backend — application logic

* Database — stores user data

You could start each container manually with separate `docker run` commands. But managing ports, networks, volumes, environment variables, and restarts becomes difficult.

Docker Compose solves this: define your application in one YAML file and manage the services together.

### 1\. How Docker Compose works

compose.yaml

Defines services, networks, volumes and configuration

docker compose up

Frontend

Backend

Database

Compose creates and manages the services and their shared application network.

Hinglish: `compose.yaml` mein poori application ki configuration likho. Ek command se services start karo aur ek command se stop/remove karo.

### 2\. Essential Compose commands

| Command                        | Purpose                                                 |
| ------------------------------ | ------------------------------------------------------- |
| docker compose up -d           | Create/start services in the background                 |
| docker compose ps              | Show services and their status                          |
| docker compose logs            | View service logs                                       |
| docker compose logs -f         | Follow logs live                                        |
| docker compose exec SERVICE sh | Open a shell in a running service                       |
| docker compose stop            | Stop services without removing them                     |
| docker compose down            | Stop and remove Compose-managed containers and networks |
| docker compose down -v         | Also remove Compose-managed named and anonymous volumes |

Warning: `down -v` can delete persistent data stored in those volumes. Don't use it casually.

### 3\. Hands-on project — Website + Redis

We'll run a small web application and a Redis service using Compose. Redis is an in-memory data store; here, we'll use it to demonstrate communication between containers.

### Step 1 — Check Docker Compose

Run:

```bash
docker compose version
```

If a version appears, continue. If not, Docker Compose may not be installed or available in your Docker setup.

### Step 2 — Create a project folder

```bash
mkdir compose-lab
cd compose-lab
```

Create the application file:

```bash
nano app.py
```

Paste this Python code:

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.request import urlopen
import os

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        try:
            with urlopen(
                "http://redis:6379", timeout=2
            ) as response:
                result = response.read().decode()
        except Exception:
            result = "Web app is running; Redis HTTP check is not applicable."

        self.send_response(200)
        self.end_headers()
        self.wfile.write(
            b"Hello from the Compose web container!"
        )

server = HTTPServer(("0.0.0.0", 8000), Handler)
print("Web app running on port 8000")
server.serve_forever()
```

This app serves a simple HTTP response. We don't need the Redis check in this example, so you can simplify the code to this cleaner version instead:

```python
from http.server import BaseHTTPRequestHandler, HTTPServer

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header("Content-Type", "text/plain")
        self.end_headers()
        self.wfile.write(
            b"Hello from the Compose web container!"
        )

HTTPServer(("0.0.0.0", 8000), Handler).serve_forever()
```

Save in Nano: Ctrl+O, Enter, then Ctrl+X.

### Step 3 — Create a Dockerfile

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

### Step 4 — Create `compose.yaml`

```bash
nano compose.yaml
```

Paste:

```yaml
services:
  web:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - redis

  redis:
    image: redis:7-alpine
```

Save and exit.

### Step 5 — Start the application

From the `compose-lab` directory, run:

```bash
docker compose up -d --build
```

Compose will build the web image, download the Redis image if needed, and create/start both services.

Check status:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs
```

### Step 6 — Test the web application

Open this address in your browser:

`http://localhost:8000`

Expected response:

```text
Hello from the Compose web container!
```

You can also test from the terminal:

```bash
curl http://localhost:8000
```

Important: The web app and Redis are on the same Compose network, so the web service can reach Redis using the hostname `redis`. Our current app doesn't actually connect to Redis; we'll add real service communication in a later project.

### 4\. Understand every line of `compose.yaml`

| YAML setting | Meaning                                                 |
| ------------ | ------------------------------------------------------- |
| services:    | Lists the application's services                        |
| web:         | Names the web service                                   |
| build: .     | Builds an image from the current directory's Dockerfile |
| ports:       | Publishes a host port to a container port               |
| "8000:8000"  | Host port 8000 → container port 8000                    |
| depends_on:  | Controls basic service startup ordering                 |
| redis:       | Names the Redis service                                 |
| image:       | Specifies the image to use                              |

By default, Compose creates a network for the project and attaches its services. Service names such as `redis` can be used as hostnames on that network.

Exam/interview trap: Basic `depends_on` controls startup order, but it doesn't necessarily mean the dependency is ready to accept requests. Health checks and readiness-aware configuration may be needed.

### 5\. Environment variables and persistent storage

Here's a common Compose pattern for a service that needs configuration:

```yaml
services:
  web:
    build: .
    environment:
      APP_ENV: development
    ports:
      - "8000:8000"
```

`APP_ENV` becomes an environment variable inside the container. Your application must read it for it to affect behavior.

For persistent data, a database service often uses a named volume:

```yaml
services:
  database:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: local-dev-password
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

This is a configuration example, not a production-ready database setup. For production, don't commit passwords into YAML; use appropriate secrets management and secure database configuration. Use a supported image version and backup plan.

### 6\. Troubleshooting

| Problem                               | First check                                 |
| ------------------------------------- | ------------------------------------------- |
| Port already in use                   | Change the host side, e.g. "8001:8000"      |
| Container exited                      | docker compose ps -a                        |
| Application errors                    | docker compose logs web                     |
| Invalid YAML                          | Check indentation; YAML uses spaces         |
| Service can't resolve another service | Check both are attached to the same network |
| Image build fails                     | Review the Dockerfile and build output      |

### 7\. Cleanup

From the project directory:

```bash
docker compose down
```

This removes the project's containers and default network, but retains named volumes unless you explicitly remove them.

You can keep the project files to practice again.

### 8\. Mini-test

1. What is Docker Compose mainly used for?

A. Editing images

B. Defining and managing multi-container applications

C. Replacing Linux

D. Creating only virtual machines

2. Which command starts services in the background and builds images when needed?

A. docker compose down

B. docker compose logs

C. docker compose up -d --build

D. docker volume rm

3. In a Compose network, how can the web service normally reach the Redis service?

A. Using hostname redis

B. Using localhost from the web container

C. Only through a public IP

D. It cannot connect

4. What does \`depends_on\` guarantee by itself?

A. The dependency is fully ready

B. The dependency is healthy forever

C. Basic startup ordering, not necessarily readiness

D. That Redis data is backed up

5. What is the risk of \`docker compose down -v\`?

A. It always deletes source code

B. It can remove Compose-managed volumes and their data

C. It only shows logs

D. It upgrades all images

Submit answers

### Module 5 completion checklist

* Created `Dockerfile` and `compose.yaml`.

* Started two services with `docker compose up -d --build`.

* Tested the web app on `localhost:8000`.

* Checked service status and logs.

* Understand services, ports, networking, and volumes in Compose.

Next module: Docker security and best practices — non-root users, image size, secrets, vulnerabilities, and production-ready container habits.
