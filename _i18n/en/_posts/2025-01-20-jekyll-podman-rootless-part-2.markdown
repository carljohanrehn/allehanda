---
layout: post
title:  "Running Jekyll in a Rootless Podman Container: Part 2 - Building and Running"
date: 2025-01-20 10:42:27 +0000
description: "The second part of our series on using Podman rootless containers with Jekyll. We will focus on building the container image and running the container using different methods."
tags: [Containers, Development, Jekyll, Podman, Rootless]
categories: [fries-al-folio]
---

In the second installment of our series, we’ll focus on building a Jekyll container image and running it using rootless Podman. If you haven’t already, consider reviewing [Part 1](https://carljohanrehn.github.io/allehanda/blog/2025/jekyll-podman-rootless-part-1/), where we set up the foundation for running Jekyll using containers.

This guide demonstrates building the image and running the container from the folder:
```text
~/my-pak/cross-python-development/torch-app-docker/jekyll/docker/fries-al-folio/
```
The file `Containerfile-pypak-jekyll-srv` will be our main point of reference.

---

## Preparing the Containerfile

The `Containerfile-pypak-jekyll-srv` is the configuration file for building the Jekyll container image. Let’s break it down:

### `Containerfile` Code Example

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

### Explanation

1. **Base Image**:
   ```dockerfile
   FROM pypak-jekyll
   ```
   It extends the base image `pypak-jekyll` (reviewed in Part 1).

2. **Locale/Timezone Adjustments**:  
   Locale (`sv_SE.UTF-8`) and Timezone (`Europe/Stockholm`) are configured to ensure compatibility with Swedish formats.

   ```dockerfile
   RUN sed -i '/sv_SE.UTF-8/s/^# //g' /etc/locale.gen && \
       locale-gen
   ENV LANG sv_SE.UTF-8
   ENV LANGUAGE sv_SE:sv
   ENV LC_ALL sv_SE.UTF-8
   ENV TZ="Europe/Stockholm"
   ```

3. **Non-root User**:
   Switching to an `app` user after managing administrative actions ensures better security.

   ```dockerfile
   USER app
   ```

4. **Ruby Environment**:  
   The `GEM_HOME` and updated `PATH` allow the app user to install and utilize Ruby gems, including `jekyll` and `bundler`.

   ```dockerfile
   ENV GEM_HOME="/home/app/gems"
   ENV PATH="/home/app/gems/bin:${PATH}"
   RUN gem install jekyll bundler
   ```

5. **Directories and Dependencies**:  
   Prepares directories for the Jekyll project and uses Bundler to resolve dependencies.

   ```dockerfile
   RUN mkdir /home/app/srv
   RUN mkdir /home/app/srv/jekyll
   ADD Gemfile /home/app/srv/jekyll
   WORKDIR /home/app/srv/jekyll
   ENV BUNDLE_GEMFILE=$WORKDIR
   RUN /home/app/gems/bin/bundle install
   ```

---

## Building the Container Image

Once the `Containerfile` is ready, run the following command to build your container image:

```bash
podman image build --no-cache --format docker -f Containerfile-pypak-jekyll-srv -t pypak-jekyll-srv .
```

- **`--no-cache`**: Ensures no cached layers are reused.
- **`--format docker`**: Uses Docker-compatible formatting.
- **`-f`**: Specifies the `Containerfile` to use.
- **`-t pypak-jekyll-srv`**: Tags the resulting image as `pypak-jekyll-srv`.

---

## Running the Container

With the container image built, there are several methods to run it based on your requirements.

### 1. Basic Container Run

For a quick inspection of the container environment, use the following:

```bash
podman container run -it --rm pypak-jekyll-srv
```

This command runs the container interactively (`-it`), removes it automatically after exiting (`--rm`), and uses the `pypak-jekyll-srv` image.

### 2. Volume Mapping

To work with your Jekyll files stored on the host, use volume mapping:

```bash
podman container run -it --rm -v .:/home/app/torch-app:Z pypak-jekyll-srv
```

- **`-v`**: Maps the current directory (`.`) on your host to `/home/app/torch-app` inside the container.
- **`:Z`**: Ensures proper SELinux permissions.

This setup allows seamless sharing of project files between the host and the container.

### 3. Running a Jupyter Notebook

To run a Jupyter Notebook server from the container, utilize port mapping:

```bash
podman container run -it -p 8888:8888 pypak-jekyll-srv
```

- **`-p 8888:8888`**: Maps host port `8888` to the same port inside the container.
- This command allows you to access the notebook at [http://localhost:8888](http://localhost:8888).

---

## Conclusion

In this post, we discussed building a container image for Jekyll and explored multiple ways to run and use it. By now, you should have a working image and be capable of running it interactively, with mapped volumes, or even hosting a Jupyter Notebook.

In the [next part](https://carljohanrehn.github.io/allehanda/blog/2025/jekyll-podman-rootless-part-3/), we’ll focus on testing and debugging your Jekyll site inside a container. We’ll also explore using `Podman Compose` to streamline your container workflows.

*Happy containerizing!*

---

*This is Part 2 of a series on running Jekyll with rootless Podman containers.*
