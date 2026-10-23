### Docker Module 4 — Storage, Volumes & Bind Mounts

Beginner → Junior DevOps | Simple English + Hinglish | Hands-on lab

In Module 3, you learned how containers communicate. Now, let’s solve another important problem:

What happens to your data when a container is deleted?

By default, files written inside a container’s writable layer are tied to that container. If you remove the container, those files are removed with it. To keep data safe beyond a container’s lifetime, use Docker storage mechanisms.

### 1\. Understand the problem

Imagine you run a database inside a container and save 1,000 customer records.

Database container

1,000 customer records

Container removed

Data in container's writable layer is lost

But data stored in a separate Docker volume can survive container removal.

Hinglish: Container ko temporary samjho. Agar data bhi container ke andar hi rakha, toh container delete karne par data ja sakta hai. Important data ko volume mein rakho.

### 2\. Three Docker storage types

![Download HD Local Named Volumes Can Be Shared Between Containers, - Docker Volume Mount Transparent PNG Image - NicePNG.com](https://images.openai.com/static-rsc-4/o6dIUROAau8Bea4u5ijLkKDvnoIS7D_yINagojJMfkjQaIHrFyvQ1085id7oLiVCFURZLLshOGY2dRzCPpWgVillRZY1QoUbI8vG9GSBK1ibrTVI44hNtEojM9iJs_HBIyVXkwGBgVXBEq6vtrChwQqjCuO8a6g8ZqBPJddubBWreGWIF7a1-JyrU5VFXdrL?purpose=fullsize)

1\. Docker volume

Docker manages the storage location. The data exists separately from the container.

Best default choice for persistent application data.

![DevOps for Data Science - 6  Demystifying Docker](https://images.openai.com/static-rsc-4/YtEv1LIi8ac3IznmlPhKrRY7bzTL5hfsK8A53Mb2e1AePCOJ03wzrxVSp9K05A3NlwK5p2uzhtjrdzU7kTuTLn10kAqtLa30AVp6FWHgPdZLdfOZKbeLEEE85KGmgIwAKjqQ_kyvCZoW5UGEhWq8rHGOVpUvv3hGz4XtsMvW7aDVKngecCqSwEBg6R52US8b?purpose=fullsize)

2\. Bind mount

Maps a specific file or directory from your computer into the container.

Useful for development, source code, and configuration files.

![How Docker Actually Works: A Deep Dive into the Internals - DEV Community](https://images.openai.com/static-rsc-4/dq7ZsyfdUiHLrfaAu2Tapg4mTot6MLgi0oI9B4gHe41UN1a5iNKpkCUR1TdruBZ-Q6c1sM0gxcsGvI5B_KYV9Rn3yTHqPPRncofP9_wEPyAnAGCQ_txr69ZeugLb-nkOoJFxrGq87VJRhL9aUBphzYdBzYuN4kudoxFFqVg_XAF54739bWUNRW_tdjGDNQvF?purpose=fullsize)

3\. Container writable layer

Files written inside the container without a separate mount.

Useful for temporary changes, not data you need to preserve.

### Quick comparison

| Feature                         | Volume        | Bind mount              | Writable layer  |
| ------------------------------- | ------------- | ----------------------- | --------------- |
| Storage managed by              | Docker        | You / host OS           | Docker          |
| Survives container removal      | Yes           | Yes, host files remain  | No              |
| Choose a host path directly     | No            | Yes                     | No              |
| Typical use                     | Database data | Source code and configs | Temporary files |
| Good for production persistence | Usually       | Sometimes               | No              |

Note: Volumes survive container removal, but they can still be deleted separately. A volume is not a backup.

### 3\. Hands-on Lab A — Create a volume

### Step 1: Create a volume

```bash
docker volume create app-data
```

List your volumes:

```bash
docker volume ls
```

You should see `app-data`.

### Step 2: Write data into the volume

```bash
docker run --name storage-demo \
  -v app-data:/data \
  alpine:3.20 \
  sh -c 'echo "Hello Docker Storage" > /data/message.txt'
```

What does `-v app-data:/data` mean?

* `app-data` — the named Docker volume.

* `/data` — the path inside the container.

The command creates the file inside the mounted volume, then the container exits.

### Step 3: Remove the container

```bash
docker rm storage-demo
```

### Step 4: Read the data using a new container

```bash
docker run --rm \
  -v app-data:/data \
  alpine:3.20 \
  cat /data/message.txt
```

Expected output:

```text
Hello Docker Storage
```

Important result: The original container is gone, but the data remains because it was stored in `app-data`.

### 4\. Hands-on Lab B — Bind mount

A bind mount lets you edit files on your computer and see them from inside the container.

This example uses your current directory in Linux/WSL.

### Step 1: Create a file on your computer

```bash
mkdir -p ~/docker-storage-lab
cd ~/docker-storage-lab
echo "Hello from my computer" > message.txt
```

### Step 2: Mount the directory into a container

```bash
docker run --rm \
  --mount type=bind,source="$(pwd)",target=/app,readonly \
  alpine:3.20 \
  cat /app/message.txt
```

Expected output:

```text
Hello from my computer
```

Here, `source="$(pwd)"` is your current host directory, and `/app` is its location inside the container.

The `readonly` setting prevents the container from writing through this mount. Edit the file on your computer, rerun the command, and you’ll see the updated content.

Windows/WSL tip: Run this lab from the Ubuntu terminal where Docker is available. If you use Docker Desktop, make sure its WSL integration is enabled for your Ubuntu distribution.

### 5\. Important commands

| Command                        | Purpose                                  |
| ------------------------------ | ---------------------------------------- |
| docker volume create app-data  | Create a volume                          |
| docker volume ls               | List volumes                             |
| docker volume inspect app-data | Inspect a volume                         |
| docker volume rm app-data      | Delete an unused volume                  |
| docker volume prune            | Remove unused volumes after confirmation |
| docker inspect CONTAINER       | Inspect mounts and container settings    |

Avoid running prune commands casually on systems with important data.

### 6\. Common interview traps

* Does deleting a container delete its named volume? No, not normally.

* Does deleting a volume delete its stored data? Yes, the volume's data is removed when the volume is deleted.

* Is a bind mount the same as a volume? No. A bind mount uses a specific host path; a volume is managed by Docker.

* Should database data be stored only in the container writable layer? No. Use persistent storage.

* Does a volume guarantee disaster recovery? No. You still need backups and a recovery plan.

* What is a read-only mount? A mount the container can read but cannot write through.

### 7\. Mini-test

1. You remove a container that used a named volume. What happens to the volume data?

A. It is always deleted

B. It normally remains

C. It moves into the image

D. It becomes a bind mount

2. Which storage type maps a specific host directory into a container?

A. Writable layer

B. Image layer

C. Bind mount

D. Bridge network

3. Where should persistent database files normally be stored?

A. Only in the container writable layer

B. In a persistent volume or suitable external storage

C. Only in container logs

D. Only in the Dockerfile

4. What does \`-v app-data:/data\` mean?

A. Mount volume app-data at /data in the container

B. Publish port data

C. Copy the volume into an image

D. Create a network named app-data

5. Is a Docker volume itself a backup?

A. Yes, always

B. Yes, if the container is running

C. No, backups must be planned separately

D. Only for Alpine containers

Submit answers

### Module 4 completion checklist

* Created a named volume and verified that its data survives container removal.

* Mounted a host directory using a bind mount.

* Can explain volume vs bind mount vs writable layer.

* Understand that persistence is not the same as backup.

Next module: Docker Compose — run a multi-container application (web app + database) using one YAML file.
