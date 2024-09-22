# Podman Architecture

## 5 Best Practices for Docker

https://devdojo.com/bobbyiliev/5-docker-best-practices-i-wish-i-knew-when-i-started

### Step 1 — Use Multi-Stage Builds for Smaller Images

Multi-stage builds are a powerful feature in Docker that help you create smaller, more efficient Docker images. This is crucial because smaller images offer several advantages:

They're faster to push and pull from registries, reducing deployment times.
They use less storage space, which can lead to cost savings in cloud environments.
They have a smaller attack surface, potentially improving security.

```
# Build stage
FROM golang:1.16 AS builder
WORKDIR /app
COPY . .
RUN go build -o main .

# Final stage
FROM alpine:latest
WORKDIR /root/
COPY --from=builder /app/main .
CMD ["./main"]
```

The first stage, labeled as builder, uses the official Go image to compile our application. This stage includes all the necessary build tools and dependencies.

The second stage starts from a minimal Alpine Linux image. We copy only the compiled binary from the builder stage using the COPY --from=builder command.

The final image contains only the compiled application and the minimal runtime requirements, resulting in a much smaller image.

By using this approach, you can significantly reduce your image size. For example, a Go application image could shrink from several hundred megabytes to just a few megabytes. This not only saves space but also reduces the time it takes to deploy your application.

### Step 2 Use .dockerignore Files

It helps to reduze the docker image

```
.git
*.md
*.log
node_modules
test
```

Step 3 — Implement Health Checks in Your Dockerfiles

Health checks are an important feature in Docker that help you make sure that your containers are not only running, but actually working as expected. They allow Docker to regularly check if your application is functioning correctly.

```
FROM nginx:latest
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1
```

Implementing health checks offers several benefits:

* It allows Docker to automatically restart containers that have become unhealthy.
* In a swarm or orchestrated environment, it enables automatic rerouting of traffic away from unhealthy containers.
* It provides a clear indicator of application health, making it easier to monitor and troubleshoot issues.

By implementing health checks, you're adding an extra layer of reliability to your containerized applications.

### Step 4 — Use Docker Compose for Local Development

Docker Compose is a tool for defining and running multi-container Docker applications. It's especially useful for local development environments where you might need to run several interconnected services.

As of recent updates to Docker, the default path for a Compose file is compose.yaml (preferred) or compose.yml in the working directory. Docker Compose also supports docker-compose.yaml and docker-compose.yml for backwards compatibility. If both files exist, Compose prefers the canonical compose.yaml.

```
version: '3'
services:
  web:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - .:/code
    environment:
      - DEBUG=True
  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_PASSWORD=secret
```

To run this multi-container setup, you simply use: `docker compose up`

Docker Compose offers several advantages for local development:

It allows you to define your entire application stack in a single file.

* You can start all services with a single command.
* It provides an isolated environment that closely mimics production.
* It's easy to add or remove services as your application evolves.

By using Docker Compose, you can significantly simplify your development workflow and ensure consistency across your team.

### Step 5 — Be Cautious with the Latest Tag

While using the latest tag might seem convenient, it can lead to unexpected issues and make your builds less reproducible. Here's why you should be cautious with the latest tag:

* Unpredictability: The latest tag doesn't refer to a specific version. It usually points to the most recent version, which can change without warning.
* Lack of consistency: Different team members or environments might pull the latest image at different times, potentially getting different versions.
* Difficulties in debugging: If an issue arises, it's harder to pinpoint which exact version of the image is causing the problem.

Instead of using latest, it's better to specify exact versions in your Dockerfile or compose file. For example:

```
FROM node:18.20.4
```

or in compose file

```
services:
  db:
    image: postgres:13.3
```

### Implement Regular Security Scans

One powerful tool for this purpose is Docker Scout, which is integrated directly into Docker Desktop and the Docker CLI. Here's how you can use it:

First, ensure you have the latest version of Docker installed.

To scan an image, use the following command:

```
docker scout cve <image_name>
```

For example:

```
docker scout cve nginx:latest
```

This command will provide a detailed report of any known vulnerabilities in the image, including severity levels and available fixes.

You can also integrate Docker Scout into your CI/CD pipeline to automatically scan images before deployment. Here's an example of how you might do this in a GitHub Actions workflow:

```
name: Docker Image CI

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Build the Docker image
      run: docker build . --file Dockerfile --tag my-image:$(date +%s)
    - name: Scan the Docker image
      run: docker scout cve my-image:$(date +%s)
```
