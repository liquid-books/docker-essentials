---
title: "Chapter 4: Orchestrating with Docker Compose"
subtitle: "Scaling Multi-Container Applications with Ease"
short_title: "4: Docker Compose"
description: "An in-depth look at multi-container applications, docker-compose.yml syntax, service dependencies, scaling, and development workflows."
label: ch04-compose
tags: [docker, compose, orchestration, devops, multi-container]
exports:
  - format: pdf
---

# Chapter 4: Orchestrating with Docker Compose

:::{figure} ../images/ch04-compose-infographic.png
:label: fig-ch04-compose-infographic
:alt: Professional infographic of Docker Compose architecture and workflow.
:width: 80%
:align: center

Docker Compose acts as the conductor for your containerized services, allowing you to define, manage, and scale multi-container applications using a single YAML file.
:::

## Introduction

In the previous chapter, we learned how to manage individual containers. However, modern applications are rarely just a single container. A typical web application might consist of a frontend, a backend API, a database, and a caching layer like Redis. 

Managing these using individual `docker run` commands is error-prone, difficult to document, and nearly impossible to scale. This is where **Docker Compose** comes in.

## What is Docker Compose?

Docker Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application's services. Then, with a single command, you create and start all the services from your configuration.

### The Three-Step Workflow

1.  Define your app's environment with a **Dockerfile** so it can be reproduced anywhere.
2.  Define the services that make up your app in **docker-compose.yml** so they can be run together in an isolated environment.
3.  Run `docker-compose up` and Compose starts and runs your entire app.

## The Anatomy of `docker-compose.yml`

The `docker-compose.yml` file is the blueprint for your application. It uses a declarative syntax to describe the desired state of your environment.

### Core Sections

- **version**: The version of the Compose file format (e.g., "3.8").
- **services**: The individual containers that make up your application.
- **networks**: Custom networks to facilitate communication between services.
- **volumes**: Persistent storage shared across services.

### Example Configuration

```yaml
version: "3.8"
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    environment:
      FLASK_ENV: development
    depends_on:
      - db
  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: example

volumes:
  postgres_data:
```

:::{note}
In recent versions of Docker (v2+), the `docker compose` command is integrated into the Docker CLI, replacing the standalone `docker-compose` tool. Both are widely used, but the integrated version is the current standard.
:::

## Service Dependencies and `depends_on`

In a multi-container setup, the order of startup often matters. A web application cannot function if the database it relies on isn't ready yet. The `depends_on` attribute expresses these dependencies.

### Readiness vs. Liveness

While `depends_on` ensures that the `db` container *starts* before the `web` container, it does **not** wait for the database service to be "ready" (i.e., accepting connections). 

:::{tip}
To ensure a service is fully ready, use **Healthchecks** in your `docker-compose.yml` combined with the `condition: service_healthy` check in `depends_on`.
:::

## Scaling and Management

One of the most powerful features of Docker Compose is the ability to scale services horizontally.

### Scaling Services

If you need more processing power for your web tier, you can scale that specific service:

```bash
docker compose up --scale web=3 -d
```

This command will start three instances of the `web` service. Note that this requires your configuration to handle port conflicts (e.g., by not mapping a specific host port or using a load balancer).

### Common Compose Commands

| Command | Description |
| :--- | :--- |
| `docker compose up` | Create and start containers. |
| `docker compose down` | Stop and remove containers, networks, and images. |
| `docker compose ps` | List containers managed by Compose. |
| `docker compose logs -f` | View output from all services in real-time. |
| `docker compose build` | Rebuild images defined in the file. |

## Development Workflows with Compose

Docker Compose is an essential tool for local development. It allows developers to spin up a "production-like" environment on their laptops with a single command.

### Bind Mounts for Live Reloading

By using bind mounts, you can map your local source code into the container. This enables "Live Reloading" or "Hot Module Replacement" (HMR). When you save a file on your host machine, the change is instantly reflected inside the running container.

:::{figure} ../images/ch04-dev-workflow.png
:label: fig-ch04-dev-workflow
:alt: Diagram of a developer workflow using Docker Compose and bind mounts.
:width: 80%
:align: center

The development workflow is accelerated by using Compose to manage auxiliary services (DB, Cache) while the app code is hot-reloaded via bind mounts.
:::

## Summary

Docker Compose simplifies the management of multi-container applications. By moving from imperative `docker run` commands to a declarative `docker-compose.yml`, you gain reproducibility, documentation, and a powerful tool for both development and orchestration.

### Discussion Questions
1. How does Docker Compose handle networking between services by default?
2. What are the limitations of `depends_on` for complex startup sequences?
3. In what scenarios would you use `docker compose build` instead of just `up`?

### Exercises
1. **Exercise 1**: Create a `docker-compose.yml` that runs a WordPress site and its MySQL database. Ensure the database password is passed via environment variables.
2. **Exercise 2**: Modify the previous exercise to use a named volume for the WordPress uploads and another for the MySQL data.
3. **Exercise 3**: Use the `links` or `depends_on` property and demonstrate service discovery by `exec`-ing into one container and pinging another by its service name.

:::{dropdown} Solution to Exercise 1 (Snippet)
```yaml
version: '3.8'
services:
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: examplepassword
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8000:80"
```
:::

## Glossary
- **YAML**: Yet Another Markup Language (or YAML Ain't Markup Language), a human-readable data serialization standard.
- **Service**: A definition of a container to be run in a multi-container environment.
- **Orchestration**: The automated configuration, management, and coordination of computer systems and software.
- **Declarative**: A programming paradigm that expresses the logic of a computation without describing its control flow (the "what" vs. the "how").
- **Hot Reloading**: The ability to update a running application with new code without restarting the entire process.
