### Docker Module 3 — Docker Networking

Beginner → Junior DevOps | Simple English + Hinglish | Hands-on lab

In Module 2, you learned how to build your own Docker image. Now you’ll learn how containers communicate with each other and with the outside world.

Simple example: Your web application runs inside one container, and your database runs inside another. Networking allows the application to connect to the database.

### 1\. What is Docker networking?

Docker networking connects containers to each other and, when needed, to external networks.

Your browser

[http://localhost:8080](http://localhost:8080?utm_source=chatgpt.com)

Published port: 8080 → 80

Web container

Nginx listens on port 80

Later, we'll connect this web container to a separate application or database container.

Hinglish: Container ke andar application apne port par run karti hai. Docker networking decide karta hai ki doosre containers ya outside systems us tak kaise pahunch sakte hain.

### 2\. Docker network types

The main network drivers you should know:

| Network | Simple meaning                                     | Common use                                               |
| ------- | -------------------------------------------------- | -------------------------------------------------------- |
| bridge  | Containers communicate on a Docker-managed network | Most single-host container applications                  |
| host    | Container shares the host’s network namespace      | Special cases where host networking is needed            |
| none    | No normal external network connectivity            | Isolation or specialized tasks                           |
| overlay | Connects containers across multiple Docker hosts   | Multi-host Docker Swarm                                  |
| macvlan | Gives a container its own MAC address on a network | Legacy or network appliances needing direct LAN presence |

For junior DevOps, focus first on `bridge`, custom bridge networks, port publishing, and container-to-container communication.

### 3\. Default bridge vs custom bridge

### Default bridge

Container A

Container B

Containers can communicate using IP addresses, but automatic container-name DNS resolution is not provided by the default bridge.

### Custom bridge

Container A

Container B

Containers can discover each other by container name using Docker's embedded DNS.

Exam/interview tip: For applications with multiple containers on one Docker host, a user-defined bridge network is usually the better choice.

### 4\. Hands-on lab: Connect two containers

We’ll create a custom bridge network and verify that one container can reach another by its name.

### Step 1 — Create a network

Run in your Ubuntu/WSL terminal:

```bash
docker network create devops-net
```

Check it:

```bash
docker network ls
```

You should see `devops-net` in the list.

### Step 2 — Start a web container on that network

```bash
docker run -d --name devops-web --network devops-net nginx:alpine
```

Check that it is running:

```bash
docker ps
```

### Step 3 — Test communication by container name

Run a temporary Alpine container on the same network:

```bash
docker run --rm --network devops-net alpine:3.20 wget -qO- http://devops-web
```

Expected result: HTML output from the Nginx welcome page.

Why does this work?

* Both containers are connected to `devops-net`.

* Docker's embedded DNS resolves `devops-web` to the web container.

* Nginx listens on port `80` inside its container.

* We don't need to publish port 80 to the host for this container-to-container test.

Successful result

If you receive HTML, container-to-container networking and name resolution are working.

### Step 4 — Publish a port to your computer

Stop and remove the first web container:

```bash
docker rm -f devops-web
```

Now run it with a published port:

```bash
docker run -d --name devops-web --network devops-net -p 8080:80 nginx:alpine
```

Open `http://localhost:8080` in your browser.

| Port | Meaning                      |
| ---- | ---------------------------- |
| 8080 | Port on your computer (host) |
| 80   | Port inside the container    |

Remember: `-p 8080:80` means host port : container port.

### Step 5 — Inspect and clean up

Inspect the network:

```bash
docker network inspect devops-net
```

Clean up the lab:

```bash
docker rm -f devops-web
docker network rm devops-net
```

The temporary Alpine container was automatically removed because we used `--rm`.

### 5\. Docker networking commands to remember

| Command                                  | Purpose                         |
| ---------------------------------------- | ------------------------------- |
| docker network ls                        | List networks                   |
| docker network create NAME               | Create a network                |
| docker network inspect NAME              | View network details            |
| docker network connect NAME CONTAINER    | Attach a container to a network |
| docker network disconnect NAME CONTAINER | Detach a container              |
| docker network rm NAME                   | Remove a network                |

### 6\. Common mistakes and interview traps

* Container port vs host port: A container can listen on port 80 without making it accessible from your computer. Publishing with `-p` exposes a host port.

* Different networks: Containers on separate isolated bridge networks generally cannot communicate directly through those networks.

* Container IPs: Avoid hardcoding container IPs; they can change. Use service/container names on a user-defined bridge.

* `EXPOSE` vs `-p`: `EXPOSE 80` documents the intended port; it does not publish the port.

* `localhost` inside a container: It refers to that same container, not another container. Use the other container's name on a shared custom bridge network.

### 7\. Mini-test

1. Which network is generally best for two containers communicating on one Docker host?

A. host

B. none

C. User-defined bridge

D. overlay is always required

2. What does \`-p 8080:80\` mean?

A. Container 8080 → host 80

B. Host 8080 → container 80

C. Both ports are host ports

D. It exposes port 80 without publishing it

3. Two containers share a user-defined bridge. How can one normally reach the other?

A. By using its container name

B. Only by its public IP

C. Only after installing Kubernetes

D. It cannot communicate

4. What does \`EXPOSE 80\` do by itself?

A. Opens port 80 on the internet

B. Publishes host port 80

C. Documents the intended container port

D. Creates a bridge network

5. Inside a container, what does \`localhost\` refer to?

A. The Docker host in every case

B. That same container

C. Every container on the network

D. The public internet

Submit answers for grading

Your completion goal: Run the networking lab successfully, explain `bridge` vs `host`, and understand `-p host:container`. After that, the next module is Docker storage: volumes, bind mounts, and persistence.
