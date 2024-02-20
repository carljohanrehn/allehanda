---
layout: post
title:  Jekyll Podman Rootless Part 2
date: 2023-04-15 11:54:27
description: 
tags: Jekyll, Podman
categories: fries-al-folio
---

---

- Location: ~/my-pak/cross-python-development/torch-app-docker/jekyll/docker/fries-al-folio/
- File: Containerfile-pypak-jekyll-srv

---

Let's create a container file:

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

Now, use Podman to build an image. Thus,

```bash
podman image build --no-cache --format docker -f Containerfile-pypak-jekyll-srv -t pypak-jekyll-srv .
```

We can create a container as follows:

```bash
podman container run -it --rm pypak-jekyll-srv
```

Also, if we need to map a volume, use

```bash
podman container run -it --rm -v .:/home/app/torch-app:Z pypak-jekyll-srv
```

It's also possible to create a Jupyter notebook. Hence,

```bash
docker container run -it -p 8888:8888 pypak-jekyll-srv
```

