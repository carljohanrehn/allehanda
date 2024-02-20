---
layout: post
title:  Jekyll Podman Rootless Part 3
date: 2023-04-15 11:58:48
description: 
tags: Jekyll, Podman
categories: fries-al-folio
---

---

- Location: ~/my-pak/cross-python-development/torch-app-docker/jekyll/docker/fries-al-folio/
- File: docker-compose-project-podman.yml

---

```bash
podman container rm al-folio-website
```

```bash
podman exec -it 13ab0f32ffe0 /bin/bash
```

```bash
podman container run -it 8f97b35e5316 /bin/bash
```

```bash
podman-compose -f docker-compose-project-podman.yml up
```

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
      - ../../..:/home/app/project:Z  # Note: PyCharm maps '.' to /opt/project/ with environment variable IDE_PROJECT_ROOTS
      - .:/home/app/srv/jekyll:Z
#    environment:
#      - JEKYLL_ROOTLESS=1  # https://github.com/envygeeks/jekyll-docker/blob/master/README.md
```

