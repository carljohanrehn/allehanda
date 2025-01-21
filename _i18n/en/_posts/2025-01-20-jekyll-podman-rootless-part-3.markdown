---
layout: post
title: "Running Jekyll in a Rootless Podman Container: Part 3 - Reproducible Development and Testing"
date: 2025-01-20 10:53:48
description: "The final part of our series on using Podman rootless containers with Jekyll. This post focuses on testing and running a reproducible server using `podman-compose`."
tags: [Containers, Development, Jekyll, Podman, Rootless]
categories: [fries-al-folio]
---

Welcome to the third and final part of our series on running Jekyll in a rootless Podman container! In this post, we’ll focus on creating a reproducible workflow for testing and development using **`podman-compose`**. This approach simplifies the process of managing containers and makes it easier to build, test, and debug your Jekyll projects.

If you’re new to this series, be sure to catch up on [Part 1](https://carljohanrehn.github.io/allehanda/blog/2025/jekyll-podman-rootless-part-1/) and [Part 2](https://carljohanrehn.github.io/allehanda/blog/2025/jekyll-podman-rootless-part-2/), where we covered the basics of rootless Podman environments and file ownership considerations. Let’s dive in!

---

## Why Use `podman-compose`?

In previous posts, we manually used Podman commands to run and stop containers. While this works, managing containers through a **`podman-compose`** workflow offers several benefits:

- **Reproducibility**: Ensures the same environment every time you spin up your containers.
- **Ease of Use**: A single command can bring up or tear down the entire development environment.
- **Scalability**: Quickly add other services to your environment if needed.

To show how this works, we’ll utilize the following `docker-compose`-compatible file: **`docker-compose-project-podman.yml`**.

---

## Understanding the Compose File

Here’s an example `docker-compose-project-podman.yml` used for our Jekyll container:

```yaml
version: "3"

services:
  jekyll:
    image: pypak-jekyll-srv
    container_name: al-folio-website
    command: bash -c "
      rm -f Gemfile.lock
      && bundler exec jekyll serve --watch --port=8080 --host=0.0.0.0 --livereload --verbose --trace"
    ports:
      - 8080:8080
    volumes:
      - ../../..:/home/app/project:Z
      - .:/home/app/srv/jekyll:Z
```

### Key Sections of the Compose File:

1. **Version**:  
   The `version: "3"` line specifies the Docker Compose file format.

2. **Services**:  
   Defines our containerized tasks, which in this case is the **`jekyll`** service.

3. **Image**:  
   The image **`pypak-jekyll-srv`** contains our Jekyll runtime.

4. **Command**:  
   This shell command ensures a clean state (`rm -f Gemfile.lock`) and runs the Jekyll server with options like:
    - `--watch`: Watches for file changes.
    - `--port=8080`: Exposes the server on port 8080.
    - `--host=0.0.0.0`: Opens the server to all interfaces.
    - `--livereload`: Enables automatic browser reloads for a better dev experience.
    - `--verbose` and `--trace`: Helpful for debugging.

5. **Ports**:  
   Maps port `8080` inside the container to port `8080` on your local machine.

6. **Volumes**:  
   Mounts host directories into the container for easy file access:
    - `../../..:/home/app/project:Z`: Maps a higher-level directory to `/home/app/project` in the container.
    - `.:/home/app/srv/jekyll:Z`: Maps the current directory to `/home/app/srv/jekyll`.

---

## Running Jekyll with `podman-compose`

### Step 1: Start the Service

Use this command to spin up the Jekyll container:

```bash
podman-compose -f docker-compose-project-podman.yml up
```

Once the container is running, you can view your Jekyll website locally by navigating to:

```text
http://localhost:8080
```

The Jekyll server is now live and automatically updates when changes are made to your files.

### Step 2: Stopping the Service

To shut down the service cleanly, run:

```bash
podman-compose -f docker-compose-project-podman.yml down
```

This command stops and removes the container.

---

## Debugging and Testing

While using `podman-compose` is convenient, you’ll occasionally need to debug or test configurations. Here are a few useful commands:

### Access the Running Container

You can directly interact with the container by opening a shell session inside it:

```bash
podman exec -it al-folio-website /bin/bash
```

This provides a terminal inside the container, allowing you to inspect files or run Jekyll commands directly.

### Testing the Container Manually

If you’d like to test the container in isolation, you can run a temporary instance with:

```bash
podman container run -it --rm pypak-jekyll-srv /bin/bash
```

This creates a disposable container, which is destroyed after you exit.

### Listing Active Containers

Verify which containers are currently running with:

```bash
podman ps
```

### Removing Stuck Containers

If a container fails to stop properly, you can forcibly remove it using:

```bash
podman container rm al-folio-website
```

---

## Recap

In this post, we discussed how to use `podman-compose` to manage and test your Jekyll development workflow. Here's a quick summary of what we covered:

1. **Getting Started**: We explored the benefits of `podman-compose` for reproducible environments.
2. **Compose File**: Broke down the essential components of a `docker-compose`-compatible file for Jekyll.
3. **Running and Debugging**: Showed how to start, stop, and debug your Jekyll container.

By using `podman-compose`, you can simplify container management and ensure consistency across environments, making your workflow more efficient.

---

## Series Wrap-Up

With this third installment, we’ve completed our series on running Jekyll in rootless Podman containers:

- [**Part 1**](https://carljohanrehn.github.io/allehanda/blog/2025/jekyll-podman-rootless-part-1/): Setting up Podman and running rootless containers.
- [**Part 2**](https://carljohanrehn.github.io/allehanda/blog/2025/jekyll-podman-rootless-part-2/): Managing file permissions and user mapping.
- **Part 3** (this post): Automating workflows with `podman-compose`.

We hope this series inspires you to experiment with containerized workflows for your development projects. If you have questions or would like to share your own use case, feel free to leave a comment below.

Happy coding, and good luck with your projects!
