---
title: "Docker: A Beginner-Friendly Guide"
tags: [docker, containers, linux, software]
lang: en
---

# Docker: A Beginner-Friendly Guide

Docker lets you run applications in isolated environments called [containers](#container). Instead of installing every application directly on your system, you can package it together with its dependencies and run it in a predictable environment.

This makes Docker useful for development, self-hosting, testing, and running services on servers.

## How Docker works

Docker uses several important concepts:

* An [image](#image) is a template used to create containers.
* A [container](#container) is a running instance of an image.
* A [volume](#volume) stores data outside the container's writable filesystem.
* [Docker Compose](#docker-compose) lets you define and run multiple containers as one application.

A simple example:

```text
Docker image
     │
     ▼
  Container
     │
     ├── Application
     ├── Dependencies
     └── Configuration
```

Containers are isolated from the host system, but they still share the host's Linux kernel.

## Installing Docker

On Arch Linux, Docker is available directly from the official repositories:

```bash
sudo pacman -S docker
```

Enable and start the Docker service:

```bash
sudo systemctl enable --now docker
```

Check that Docker is working:

```bash
docker version
```

You can also run a small test container:

```bash
docker run hello-world
```

If Docker prints a welcome message, the installation is working.

### Running Docker without root

By default, Docker commands require access to the Docker daemon, which commonly means using root privileges.

You can add your user to the `docker` group:

```bash
sudo usermod -aG docker "$USER"
```

Log out and back in for the new group membership to take effect.

Then check:

```bash
docker ps
```

> **Security note:** Membership in the `docker` group effectively grants root-level control over the host. Do not add untrusted users to this group.

## Running your first container

Let's run an Nginx web server:

```bash
docker run --name my-nginx -p 8080:80 nginx
```

Docker will download the `nginx` image if it is not already available locally and start a container from it.

The `-p 8080:80` option maps port `8080` on the host to port `80` inside the container.

You can now open:

```text
http://localhost:8080
```

To see running containers:

```bash
docker ps
```

To stop the container:

```bash
docker stop my-nginx
```

To start it again:

```bash
docker start my-nginx
```

To remove it:

```bash
docker rm my-nginx
```

## Working with images

Images are usually downloaded from a [container registry](#container-registry), such as Docker Hub.

Pull an image manually:

```bash
docker pull nginx
```

List downloaded images:

```bash
docker images
```

Remove an image:

```bash
docker rmi nginx
```

You normally do not need to manually pull an image before running it. `docker run` will download it automatically if necessary.

## Container lifecycle

A container can be running, stopped, or removed.

Useful commands:

```bash
docker ps
```

Show running containers.

```bash
docker ps -a
```

Show all containers, including stopped ones.

```bash
docker stop <container>
```

Stop a running container.

```bash
docker start <container>
```

Start an existing container.

```bash
docker restart <container>
```

Restart a container.

```bash
docker rm <container>
```

Remove a stopped container.

A stopped container still exists and can usually be started again. Removing it deletes the container itself.

## Viewing logs

If an application inside a container is not behaving as expected, check its logs:

```bash
docker logs <container>
```

Follow new log messages:

```bash
docker logs -f <container>
```

This is often the first thing to check when a container starts and immediately exits.

## Executing commands inside a container

You can open a shell inside a running container:

```bash
docker exec -it <container> sh
```

Some images include Bash instead:

```bash
docker exec -it <container> bash
```

For example, you can check the filesystem or inspect configuration files from inside the container.

Avoid modifying containers manually when possible. If configuration needs to be reproducible, put it in the Dockerfile, Compose configuration, or mounted configuration files instead.

## Persistent data

Containers are designed to be disposable. Data stored only inside a container can disappear when the container is removed.

For persistent data, use a [volume](#volume).

Create a volume:

```bash
docker volume create app-data
```

Use it with a container:

```bash
docker run \
  --name my-app \
  -v app-data:/data \
  my-image
```

The `/data` directory inside the container is now backed by the Docker volume.

List volumes:

```bash
docker volume ls
```

Remove a volume:

```bash
docker volume rm app-data
```

Be careful when removing volumes: they may contain important application data.

## Bind mounts

Instead of a Docker-managed volume, you can mount a directory from the host:

```bash
docker run \
  -v "$PWD/config:/app/config" \
  my-image
```

This is useful when you want to edit configuration files directly on the host.

A [bind mount](#bind-mount) is different from a Docker volume because the data lives at a specific path on the host.

## Environment variables

Many Docker images use environment variables for configuration.

For example:

```bash
docker run \
  -e APP_ENV=production \
  -e APP_PORT=8080 \
  my-image
```

You can also load variables from a file:

```bash
docker run --env-file .env my-image
```

Do not put passwords, API keys, or other sensitive credentials into images or Git repositories.

## Docker Compose

Running one container with `docker run` is simple, but applications often consist of multiple services.

For example:

```text
Web application
      │
      ├── Backend
      ├── PostgreSQL
      └── Redis
```

[Docker Compose](#docker-compose) allows these services to be described in one YAML file.

A minimal example:

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"

  redis:
    image: redis:alpine
```

Save it as `compose.yaml`.

Start the application:

```bash
docker compose up -d
```

Stop it:

```bash
docker compose down
```

Show the service status:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs
```

Follow logs:

```bash
docker compose logs -f
```

Compose is especially useful for development environments and self-hosted applications.

## Building your own image

You can create an image using a [Dockerfile](#dockerfile).

For example:

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

Build it:

```bash
docker build -t my-python-app .
```

Run it:

```bash
docker run --rm my-python-app
```

The `--rm` option automatically removes the container after it exits.

## Keeping images up to date

Docker images do not automatically update when the image on the registry changes.

For example, if you have:

```yaml
services:
  web:
    image: nginx:latest
```

you can pull the newest image with:

```bash
docker compose pull
```

Then recreate the containers:

```bash
docker compose up -d
```

For production systems, avoid blindly using floating tags such as `latest`. Pinning a specific version makes deployments more predictable.

## Cleaning up unused resources

Docker can accumulate stopped containers, unused images, networks, and build cache.

You can inspect disk usage:

```bash
docker system df
```

Remove unused resources:

```bash
docker system prune
```

Docker will ask for confirmation before removing resources.

Be careful with aggressive cleanup commands, especially on servers. Make sure you understand what is unused before deleting it.

## Common mistakes

### Treating containers like virtual machines

Containers are not full virtual machines. They share the host kernel and are usually much lighter than traditional VMs.

### Storing important data inside containers

If an application needs persistent data, use a volume or bind mount.

### Using `latest` everywhere

A moving tag can change underneath you. Pin versions when reproducibility matters.

### Publishing every port

Only expose ports that need to be reachable from outside the container or host.

### Putting secrets into Dockerfiles

Anything included in an image can potentially be extracted from it. Keep secrets outside the image and provide them at runtime.
