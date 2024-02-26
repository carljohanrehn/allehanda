---
layout: post
title:  Jekyll Python Package Part 2
date: 2023-04-16 10:20:25
description: 
tags: Jekyll, Podman
categories: fries-al-folio
---

---

- Location: ~/my-pak/cross-python-development/torch-app-docker/
- File: docker-compose-pypak-jekyll.yml

---


```bash
# https://github.com/compose-spec/compose-spec
# https://github.com/compose-spec/compose-spec/blob/master/spec.md#volumes-top-level-element
```

```bash
docker compose -f docker-compose-pypak-jekyll.yml run publish
docker compose -f docker-compose-pypak-jekyll.yml up notebook
```

```bash
# https://github.com/containers/podman-compose
# https://www.redhat.com/sysadmin/podman-compose-docker-compose
# https://www.redhat.com/sysadmin/compose-kubernetes-podman
```

```bash
# https://linuxhandbook.com/podman-compose/
```

```bash
podman-compose -f docker-compose-pypak-jekyll.yml run publish
podman-compose -f docker-compose-pypak-jekyll.yml up notebook
```

```bash
# alias docker=podman
docker-compose -f docker-compose-pypak-jekyll.yml run publish
docker-compose -f docker-compose-pypa-jekyllk.yml up notebook
```

```yaml
version: "3.9"

services:

  publish:
    image:
      pypak-jekyll
    volumes:
      - .:/home/app/project:Z  # Note: PyCharm maps '.' to /opt/project/ with environment variable IDE_PROJECT_ROOTS

  notebook:
    image:
      pypak-jekyll
    entrypoint:
      py -m jupyter lab --no-browser --ip=0.0.0.0 --port=8888 --notebook-dir=/home/app --NotebookApp.token='' --NotebookApp.password=''
    volumes:
      - .:/home/app/project:Z
    ports:
      - 8888:8888
```
