# Chapter 2: Building Your First Image

---
short_title: Building Images
description: Learning Dockerfile instructions, image layers, and the build context.
label: ch02-building-images
exports:
  - format: pdf
---

:::{figure} ../images/ch02-infographic.png
:label: fig-ch02-building-images
:alt: Building Docker Images Infographic
:width: 80%
:align: center

The Docker build process: from Dockerfile and Build Context to a layered Image.
:::

In Docker, everything starts with an **Image**. If a container is a running instance of a process, the image is the template, the blueprint, and the snapshot of the filesystem required to run that process.

In this chapter, we will learn how to go from a simple source code file to a fully functional Docker image.

## The Dockerfile

A `Dockerfile` is a text document that contains all the commands a user could call on the command line to assemble an image.

### Basic Instructions

| Instruction | Description |
| :--- | :--- |
| `FROM` | Initializes a new build stage and sets the **Base Image**. |
| `WORKDIR` | Sets the working directory for any following instructions. |
| `COPY` | Copies new files or directories from the source to the image. |
| `RUN` | Executes commands in a new layer on top of the current image. |
| `CMD` | Provides defaults for an executing container. |

```{dropdown} Example Dockerfile for a Node.js App
```docker
# Use an official Node.js runtime as a parent image
FROM node:18-slim

# Set the working directory
WORKDIR /app

# Copy package files and install dependencies
COPY package*.json ./
RUN npm install

# Copy the rest of the application code
COPY . .

# Run the application
CMD ["node", "index.js"]
```
```

## Layers and Caching

Docker images are composed of **Layers**. Each instruction in a Dockerfile creates a new layer. This architecture is what makes Docker so fast and efficient.

### Why Layers Matter
When you change a file and rebuild your image, Docker only rebuilds the layers that have changed. It reuses the **Cache** for all previous layers.

```{important}
Order matters! Always put instructions that change frequently (like `COPY . .`) at the bottom of your Dockerfile to maximize cache hits for heavy steps like `npm install`.
```

## The Build Command

To turn your Dockerfile into an image, use the `docker build` command.

```bash
docker build -t my-app:v1.0 .
```

- `-t`: Tags your image with a name and version.
- `.`: Specifies the **Build Context** (the current directory).

::::{tab-set}
:::{tab-item} Build
`docker build -t app:latest .`
:::
:::{tab-item} List
`docker images`
:::
:::{tab-item} Run
`docker run app:latest`
:::
::::

## Summary

Building images is the core workflow of a Docker developer. By understanding how Dockerfiles translate into cached layers, you can build lean, fast, and reproducible environments for any application.
