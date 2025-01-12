---
layout: post
title:  "Running Jekyll in a Rootless Podman Container: Part 2 - Building and Running"
date: 2023-04-15 11:54:27
description: "The second part of our series on using Podman rootless containers with Jekyll. We will focus on building the container image and running the container using different methods."
tags: Jekyll, Podman, Containers, Rootless, Development
categories: fries-al-folio
---

---

In this second part of our series, we'll delve into building our Jekyll image and exploring various ways to run it as a rootless container. This builds upon the foundation we created in [Part 1](link-to-part-1). This post will guide you through the steps using the following folder:
```text
~/my-pak/cross-python-development/torch-app-docker/jekyll/docker/fries-al-folio/
```

The relevant file is named: `Containerfile-pypak-jekyll-srv`.

## Examining the Containerfile
Let's create a `Containerfile` that sets up Jekyll, installs dependencies, and prepares the container environment. This is the `Containerfile-pypak-jekyll-srv`:

```dockerfile
FROM pypak-jekyll

USER root

RUN sed -i '/sv_SE.UTF-8/s/^# //g' /etc/locale.gen && \
    locale-gen
ENV LANG sv_SE.UTF-8
ENV LANGUAGE sv_SE:sv
ENV LC_ALL sv_SE.UTF-8
ENV TZ="Europe/Stockholm"

USER app

ENV GEM_HOME="/home/app/gems"
ENV PATH="/home/app/gems/bin:${PATH}"

RUN gem install jekyll bundler
RUN mkdir /home/app/srv
RUN mkdir /home/app/srv/jekyll
ADD Gemfile /home/app/srv/jekyll

WORKDIR /home/app/srv/jekyll

ENV BUNDLE_GEMFILE=$WORKDIR
RUN /home/app/gems/bin/bundle install
```

Here's a breakdown of what this `Containerfile` does:

*   **`FROM pypak-jekyll`**: Starts from a base image named `pypak-jekyll` (we assume this was defined in the previous post).
*   **`USER root`**: Sets the user to root so that we can run commands that need root privileges, specifically to change the locale settings and install packages.
*   **Locale and Timezone Configuration**:
    *   `RUN sed -i ...`: Uncomments the `sv_SE.UTF-8` locale in the `/etc/locale.gen` file.
    *   `locale-gen`: Generates the locale.
    *   `ENV LANG ...`, `ENV LANGUAGE ...`, `ENV LC_ALL ...`: Sets environment variables for the Swedish locale.
    *   `ENV TZ="Europe/Stockholm"`: Sets the timezone to Europe/Stockholm.
*   **`USER app`**: Switches to the `app` user for all further operations. It is a common practice to not run the whole container as root.
*   **`ENV GEM_HOME ...`**: Sets up the path for Ruby gems, avoiding conflicts with other ruby installations.
*   **`ENV PATH ...`**: Updates the system's `PATH` environment variable to include the path for Ruby gems executable.
*   **`RUN gem install jekyll bundler`**: Installs Jekyll and Bundler.
*   **`RUN mkdir ...`**: Creates directories for Jekyll and source files.
*   **`ADD Gemfile ...`**: Adds the `Gemfile` which should specify jekyll dependencies.
*   **`WORKDIR /home/app/srv/jekyll`**: Sets the working directory in the container.
*   **`ENV BUNDLE_GEMFILE=$WORKDIR`**: Sets the location of the `Gemfile`.
*   **`RUN /home/app/gems/bin/bundle install`**: Installs the gems specified by the `Gemfile`.

## Building the Container Image

Now that we understand our `Containerfile`, we can build the image with the following command:

```bash
podman image build --no-cache --format docker -f Containerfile-pypak-jekyll-srv -t pypak-jekyll-srv .
```

This command creates the `pypak-jekyll-srv` image based on our `Containerfile`.

## Running the Container

Once you have built the image, it is time to run it! Let's see a few methods.

### Basic Container Run

To run a container with the image we just built, and inspect the contents of the container we can use the following command

```bash
podman container run -it --rm pypak-jekyll-srv
```

*   `-it`: Runs the container in interactive mode with a pseudo-TTY, allowing you to interact with the shell of the container.
*   `--rm`: Automatically removes the container when it stops.
*   `pypak-jekyll-srv`: The name of the image to use to run the container.

This is a basic way of running a container, but often you need to add volumes to use the contents of your host in your container.

### Mapping a Volume

To map a volume (a directory from your host into your container) you can use the following command:

```bash
podman container run -it --rm -v .:/home/app/torch-app:Z pypak-jekyll-srv
```

*   `-v .:/home/app/torch-app:Z`: Mounts the current directory (`.`) on the host to the `/home/app/torch-app` directory in the container. `:Z` ensures that the mapping is correct for SELinux labels if needed.

This command will allow you to run the jekyll server with the code in your host in the container.

### Running a Jupyter Notebook Server
You can also run a jupyter notebook server with the following command:

```bash
podman container run -it -p 8888:8888 pypak-jekyll-srv
```

*   `-p 8888:8888`: maps port `8888` from the host to the port `8888` of the container.

This will enable you to open the jupyter notebook from your local browser by going to `localhost:8888`.

## Next Steps

In this part, we built the Jekyll image and saw how to run our containers using various configurations.

In the next part of the series, we will explore how to test and debug your Jekyll site while running it inside a rootless container in a reproducible way. We will also use compose to facilitate the execution of the container.

---

*This is Part 2 of a series on using rootless Podman containers with Jekyll.*
