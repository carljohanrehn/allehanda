---
layout: post
title:  Jekyll with rootless containers
date: 2023-04-10 09:42:08
description: 
tags: Podman, docker compose, entrypoints, volumes
categories: al-folio
---

---

File: 2023-04-10-al-folio-project.markdown

---

This post describes how to build and run Jekyll images with Podman in a rootless container. Furthermore, I use docker compose file for creating services and specifying volume and entrypoints. running the containers with podman-compose. 

To build the image I use the following container file:
```bash
emacs Containerfile-pypak-jekyll-srv
```

Let's issue the build command:
```bash
podman image build --no-cache --format docker -f Containerfile-pypak-jekyll-srv -t pypak-jekyll-srv .
```

We can create a container as follows
```bash
podman container run -it --rm pypak-jekyll-srv
```

```bash
# https://github.com/containers/podman/issues/3683
# https://unix.stackexchange.com/questions/728801/host-wide-consequences-of-setting-selinux-z-z-option-on-container-bind-mounts
# https://manpages.ubuntu.com/manpages/jammy/man1/docker-container-create.1.html
# podman container run -it --rm -v .:/home/app/torch-app:Z pypak-jekyll-srv

# docker image build -f Containerfile-pypak-jekyll-srv -t pypak-jekyll-srv .
# docker container run -it pypak-jekyll-srv
# docker container run -it --rm pypak-jekyll-srv

# Jupyter notebook port mapping
# docker container run -it -p 8888:8888 pypak-jekyll-srv


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

```bash
ls -al
podman unshare ls -al /home/alpha/my-pytorch/github-pages/fries-al-folio
```

```bash
podman image build --no-cache --format docker -f Containerfile-project-sv -t al-folio-project .
```

```bash
podman image build --format docker -f Containerfile-project-sv -t al-folio-project .
```

```bash
podman unshare chown 1000:1000 -R /home/alpha/my-pytorch/github-pages/fries-al-folio
```

```bash
podman unshare ls -al /home/alpha/my-pytorch/github-pages/fries-al-folio
```

```bash
sudo chown alpha:alpha -R /home/alpha/my-pytorch/github-pages/fries-al-folio
podman unshare ls -al /home/alpha/my-pytorch/github-pages/fries-al-folio
ls -al
```

TODO set user: root  # or app?

```bash
# podman-compose -f docker-compose-project-podman.yml up
```



Use podman compose

```bash
podman container rm al-folio-website
```

```bash
podman exec...
podman exec -it 13ab0f32ffe0 /bin/bash
```

```bash
podman container  run...
podman container run -it 8f97b35e5316 /bin/bash
```

```bash
podman-compose -f docker-compose-project-podman.yml up
```

```yaml
version: "3"
# this file uses prebuilt image in dockerhub
services:
  jekyll:
#    image: al-folio-project
    image: pypak-jekyll-srv
    container_name: al-folio-website
    command: bash -c "
      rm -f Gemfile.lock
      && bundler exec jekyll serve --watch --port=8080 --host=0.0.0.0 --livereload --verbose --trace"
    ports:
      - 8080:8080
    volumes:
      - ../../..:/home/app/project:Z  # Note: PyCharm maps '.' to /opt/project/ with environment variable IDE_PROJECT_ROOTS
      - .:/home/app/srv/jekyll:Z
#    environment:
#      - JEKYLL_ROOTLESS=1  # https://github.com/envygeeks/jekyll-docker/blob/master/README.md
```



