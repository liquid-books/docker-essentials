---
title: "Chapter 3: Managing Containers"
subtitle: "Mastering the Container Lifecycle, Networking, and Storage"
short_title: "3: Managing Containers"
description: "A deep dive into the container lifecycle (run, stop, start, restart, exec, logs), environment variables, networking, and volumes."
label: ch03-lifecycle
tags: [docker, containers, lifecycle, networking, storage]
exports:
  - format: pdf
---

# Chapter 3: Managing Containers

:::{figure} ../images/ch03-lifecycle-infographic.png
:label: fig-ch03-lifecycle-infographic
:alt: Professional infographic of the Docker container lifecycle, networking, and volumes.
:width: 80%
:align: center

A comprehensive overview of the Docker container lifecycle, including execution states, network isolation, and persistent storage mechanisms.
:::

## Introduction

In the previous chapters, we introduced the concept of containerization and how to build custom images. Now, we shift our focus to the operational heart of Docker: **Managing Containers**. Understanding the container lifecycle is not just about knowing commands; it's about mastering how processes are isolated, connected, and persisted in a DevOps environment.

This chapter will guide you through the transition of a container from a static image to a running process, exploring the nuances of networking, environment configuration, and data persistence.

## The Container Lifecycle

A Docker container is a runnable instance of an image. It has a well-defined lifecycle managed by the Docker Engine. Understanding this lifecycle is crucial for debugging and orchestration.

### States of a Container

A container can exist in several states:

1.  **Created**: The container is initialized from an image but hasn't started yet.
2.  **Running**: The main process (ENTRYPOINT/CMD) is active.
3.  **Paused**: All processes in the container are suspended.
4.  **Exited**: The main process has terminated (either successfully with code 0 or with an error).
5.  **Restarting**: Docker is attempting to bring the container back to a Running state.

:::{mermaid}
graph LR
    Image -- docker create --> Created
    Created -- docker start --> Running
    Running -- docker stop --> Exited
    Exited -- docker start --> Running
    Running -- docker pause --> Paused
    Paused -- docker unpause --> Running
    Running -- docker restart --> Running
    Running -- process exit --> Exited
    Exited -- docker rm --> Deleted
:::

### Essential Lifecycle Commands

| Command | Action | Description |
| :--- | :--- | :--- |
| `docker run` | Create + Start | Pulls the image (if needed), creates a container, and starts it. |
| `docker stop` | SIGTERM then SIGKILL | Sends a termination signal to the main process. |
| `docker start` | Start Existing | Resumes an exited container without creating a new one. |
| `docker restart` | Stop + Start | Restarts a running or stopped container. |
| `docker pause` | Suspend | Uses the `freezer` cgroup to suspend all processes. |
| `docker kill` | SIGKILL | Immediately terminates the container process. |

:::{tip}
Use `docker run --rm` during development to ensure containers are automatically removed when they exit, preventing "container clutter" on your host machine.
:::

## Inspecting and Executing

Once a container is running, we often need to peek inside or interact with it.

### Logs and Monitoring

The `docker logs` command is the primary tool for debugging. By default, it retrieves the `stdout` and `stderr` of the container's PID 1.

- `docker logs <id>`: Show current logs.
- `docker logs -f <id>`: Follow the logs in real-time.
- `docker logs --tail 100 <id>`: Show only the last 100 lines.

### Executing Commands in Running Containers

The `docker exec` command allows you to run a *new* process inside an existing, running container. This is primarily used for debugging or administrative tasks.

```bash
# Open an interactive shell inside a running container
docker exec -it my_web_server /bin/bash
```

## Environment Variables and Configuration

Containers should be treated as **immutable**. Instead of hard-coding configuration inside the image, we use environment variables to configure containers at runtime. This allows the same image to be used across Development, Staging, and Production environments.

### Passing Environment Variables

You can pass variables using the `-e` flag or an `--env-file`.

```bash
# Direct pass
docker run -e DB_HOST=production.db.com -e DB_PASS=secret my_app

# Using a file
docker run --env-file .env my_app
```

:::{note}
Sensitive information like passwords should ideally be managed via Docker Secrets or an external Vault, but environment variables are the standard mechanism for general configuration.
:::

## Container Networking

Networking is what allows containers to talk to each other and the outside world.

### Default Network Drivers

- **Bridge**: The default driver. Containers on the same bridge network can communicate using IP addresses.
- **Host**: Removes network isolation between the container and the Docker host.
- **None**: Disables all networking for the container.
- **Overlay**: Used for communication between containers on different Docker hosts (Swarm/K8s).

### Port Mapping

By default, services inside a container are not accessible from the host. We use **Port Mapping** to bridge this gap.

```bash
# Map host port 8080 to container port 80
docker run -p 8080:80 nginx
```

:::{figure} ../images/ch03-port-mapping.png
:label: fig-ch03-port-mapping
:alt: Diagram showing port mapping from host machine to a Docker container.
:width: 70%
:align: center

Port mapping bridges the isolation layer, allowing external traffic to reach services running inside the container.
:::

## Data Persistence: Volumes and Mounts

Containers are ephemeral. If a container is deleted, any data written to its internal writable layer is lost. To persist data, Docker provides **Volumes** and **Bind Mounts**.

### Comparison: Volumes vs. Bind Mounts

:::{tabs}
{tab} Volumes
Managed by Docker. Stored in `/var/lib/docker/volumes/`. Best for production data persistence as they are isolated from the host's file system structure.
{tab} Bind Mounts
Map a specific path on the host to a path in the container. Excellent for local development (e.g., mapping your source code folder to the container's web root).
:::

```bash
# Creating and using a Volume
docker volume create my_data
docker run -v my_data:/app/data my_image

# Using a Bind Mount
docker run -v $(pwd):/app my_image
```

## Summary

In this chapter, we explored how to manage the lifecycle of a container, configure it via environment variables, connect it to the network, and persist its data. Mastering these four pillars—Lifecycle, Config, Networking, and Storage—is the foundation of Docker operations.

### Discussion Questions
1. Why is it preferred to use `docker stop` over `docker kill` in a production environment?
2. Explain the difference between `docker run` and `docker exec`.
3. When would you choose a Bind Mount over a Docker Volume?

### Exercises
1. **Exercise 1**: Start an Nginx container in detached mode, map it to host port 80, and then use `docker logs -f` to watch the access logs as you refresh your browser.
2. **Exercise 2**: Create a named volume `pg_data` and mount it to a PostgreSQL container at `/var/lib/postgresql/data`. Stop the container, remove it, and start a new one with the same volume to verify data persistence.
3. **Exercise 3**: Use `docker network create` to build a custom bridge network and run two containers on it. Demonstrate that they can ping each other by container name.

:::{dropdown} Solution to Exercise 3
```bash
# Create the network
docker network create my_internal_net

# Run container A
docker run -d --name app_a --network my_internal_net alpine sleep 3600

# Run container B and ping A
docker run --network my_internal_net alpine ping -c 4 app_a
```
:::

## Glossary
- **PID 1**: The first process in a container, which must handle signals like SIGTERM.
- **Ephemeral**: The characteristic of being short-lived or temporary.
- **SIGTERM**: A signal sent to a process to request its termination gracefully.
- **Bridge Network**: A software-defined network that allows containers to communicate.
- **Idempotent**: An operation that can be applied multiple times without changing the result beyond the initial application.
