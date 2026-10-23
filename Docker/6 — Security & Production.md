### Docker Module 6 — Security & Production Best Practices

Beginner → Junior DevOps | Simple English + Hinglish | Practical commands

In previous modules, you learned how to build images, run containers, configure networks, persist data, and use Docker Compose.

Now learn how to run containers securely, efficiently, and reliably — skills commonly expected in junior DevOps roles.

### 1\. The production mindset

A container running successfully does not automatically mean it is secure.

Security

Run as a non-root user, avoid exposed secrets, reduce unnecessary privileges.

Small, maintainable images

Use suitable base images, fewer unnecessary packages, and multi-stage builds where appropriate.

Reliability

Handle logs, restart behavior, health checks, and resource limits appropriately.

### 2\. Run containers as a non-root user

Linux root has extensive privileges inside its environment. Running an application as root unnecessarily increases the impact of some security issues.

### Dockerfile example

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY --chown=10001:10001 app.py .

USER 10001:10001

EXPOSE 8000

CMD ["python", "app.py"]
```

`USER 10001:10001` sets the runtime user and group IDs.

Hinglish: Application ko root user ke under chalana zaroori nahi hai. Jitni permissions required hain, utni hi do.

Important: The application must have permission to read its files and write to any directories it needs. A non-root user alone does not guarantee complete container security.

### Hands-on check

Run an Alpine container as a non-root user:

```bash
docker run --rm --user 10001:10001 alpine:3.20 id
```

Expected output will show a UID and GID of `10001`.

### 3\. Never bake secrets into images

Never put passwords, API keys, or access tokens directly into a Dockerfile.

Bad example:

```dockerfile
ENV API_KEY=my-secret-key
```

Also avoid passing secrets through Docker build arguments: build arguments can be exposed through build metadata, logs, or image history depending on how they are used.

Better approaches:

* Use a secrets manager appropriate to your environment.

* Use Docker or orchestrator secrets where supported.

* Inject secrets securely at runtime.

* Keep `.env` files containing real secrets out of Git.

A `.dockerignore` file can help prevent accidental inclusion of files in your build context:

```text
.git
.env
*.pem
*.key
__pycache__
*.pyc
```

This reduces accidental exposure but is not a substitute for proper secret management.

### 4\. Use small, trusted images

Smaller images can reduce download time, storage consumption, and the amount of software that needs patching.

| Practice                       | Why it helps                             |
| ------------------------------ | ---------------------------------------- |
| Use a suitable slim base image | Reduces unnecessary packages             |
| Pin a version intentionally    | Makes builds more predictable            |
| Use multi-stage builds         | Keeps build tools out of the final image |
| Remove unnecessary packages    | Reduces attack surface                   |
| Scan images                    | Helps detect known vulnerabilities       |

Example:

```dockerfile
FROM python:3.12-slim
```

Avoid using `latest` blindly in production because the image it points to can change over time. For stronger reproducibility, use a specific version and, where appropriate, an image digest.

Interview distinction: A smaller image is not automatically a safer image. You still need trusted sources, patching, and vulnerability scanning.

### 5\. Understand Dockerfile best practices

Compare these two patterns.

Less efficient:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY . .

RUN pip install -r requirements.txt
```

Better for build caching:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .
```

Why? If only `app.py` changes, Docker can often reuse the dependency installation layer.

The `.dockerignore` file also keeps unnecessary files out of the build context.

### 6\. Limit container privileges

A container should have only the permissions and capabilities it needs.

Useful options include:

| Option                                | Purpose                                                                                 |
| ------------------------------------- | --------------------------------------------------------------------------------------- |
| --read-only                           | Makes the container root filesystem read-only                                           |
| --cap-drop=ALL                        | Drops Linux capabilities, which may then be selectively added if required               |
| --security-opt=no-new-privileges:true | Prevents processes from gaining additional privileges through mechanisms such as setuid |
| --memory=256m                         | Sets a memory limit                                                                     |
| --cpus=0.5                            | Limits CPU usage                                                                        |

These options may require application changes. For example, an application that writes temporary files may need an appropriate writable temporary filesystem or mounted directory.

### Hands-on lab — Harden a test container

Run:

```bash
docker run --rm \
  --read-only \
  --cap-drop=ALL \
  --security-opt=no-new-privileges:true \
  --memory=128m \
  --cpus=0.5 \
  alpine:3.20 \
  sh -c 'id && echo "Container started securely"'
```

The container should print its user information and the message, then exit.

What did you apply?

* Read-only root filesystem.

* No extra Linux capabilities.

* No privilege escalation.

* Memory and CPU limits.

This is a learning example, not a universal production configuration. Real applications may need specific capabilities or writable paths.

### 7\. Health checks and restart policies

A running container does not necessarily mean its application is healthy.

Add a health check when the application supports a meaningful health endpoint or command.

Example for Nginx:

```dockerfile
FROM nginx:alpine

HEALTHCHECK --interval=30s --timeout=3s \
  --start-period=5s --retries=3 \
  CMD wget -q -O /dev/null http://127.0.0.1/ || exit 1
```

Check health with:

```bash
docker ps
docker inspect --format='{{.State.Health.Status}}' CONTAINER_NAME
```

A restart policy can help a service recover when its container process exits:

```bash
docker run -d \
  --name restart-demo \
  --restart unless-stopped \
  nginx:alpine
```

Remember:

* `no` — no automatic restart policy.

* `on-failure` — restart after a non-zero exit, optionally with a retry limit.

* `always` — restart when the container exits; Docker also attempts to restart it after daemon restarts.

* `unless-stopped` — similar to `always`, except a manually stopped container stays stopped across daemon restarts.

A restart policy does not fix a broken application. Health checks also do not automatically restart an unhealthy container under the normal Docker Engine restart policy.

### 8\. Logs and troubleshooting

Containers should generally write application logs to standard output and standard error so Docker can collect them.

```bash
docker logs CONTAINER_NAME
docker logs --tail 100 CONTAINER_NAME
docker logs -f CONTAINER_NAME
docker stats
docker inspect CONTAINER_NAME
```

| Symptom                     | First checks                                                  |
| --------------------------- | ------------------------------------------------------------- |
| Container exits immediately | docker ps -a, then docker logs                                |
| Container is unhealthy      | Inspect health-check output and application logs              |
| Memory problems             | docker stats and application metrics                          |
| Permission denied           | Check USER, file ownership, and mounted-directory permissions |
| Application unreachable     | Check ports, listening address, and networks                  |

### 9\. Docker security checklist

Use this checklist when preparing an application for a junior DevOps project.

### Security readiness

0/10

Use a trusted, maintained base image

Run the application as a non-root user where possible

Keep secrets out of Dockerfiles, images, and Git

Add a suitable .dockerignore file

Scan images for known vulnerabilities

Avoid privileged mode unless truly required

Use read-only filesystems and drop unnecessary capabilities where possible

Set appropriate CPU and memory limits

Configure meaningful health checks and restart behavior

Centralize logs and plan image updates and backups

### 10\. Mini-test

1. Why run an application as a non-root user?

A. To make the image larger

B. To reduce unnecessary privileges

C. To automatically encrypt every file

D. To avoid needing an image

2. Where should production API keys normally be stored?

A. Hardcoded in the Dockerfile

B. Committed to Git

C. In an appropriate secrets-management system

D. In a public image label

3. What does \`--read-only\` do?

A. Makes the container's root filesystem read-only

B. Makes every mounted volume read-only automatically

C. Prevents all network traffic

D. Stops the container after one request

4. Does an unhealthy status automatically trigger a normal Docker restart policy?

A. Yes, always

B. No, not by itself

C. Only if the image is small

D. Only when using EXPOSE

5. What is the main benefit of multi-stage builds?

A. They guarantee zero vulnerabilities

B. They remove the need for testing

C. They can keep build tools out of the final image

D. They automatically back up volumes

Submit answers

Next module: Docker troubleshooting, image registries, and CI/CD integration — build, tag, push, and deploy an image using a pipeline.
