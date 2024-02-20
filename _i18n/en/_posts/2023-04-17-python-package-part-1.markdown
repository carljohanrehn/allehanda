---
layout: post
title:  Python Package Part 1
date: 2023-04-17 10:44:42
description: 
tags: Jekyll Podman
categories: pypak torch-app-docker
---

---

- Location: ~/my-pak/cross-python-development/
- File: README-torch-pak-rootless

---

```bash
https://www.tutorialworks.com/podman-rootless-volumes/
```

```bash
# Linux namespace and podman unshare for rootless containers
```

```bash
# Note: user alpha on host maps to user app in container
```

```bash
ls -al /home/alpha/my-pak/torch-pak/
```

```bash
podman unshare ls -al /home/alpha/my-pak/torch-pak/
```

```bash
podman unshare chown 1000:1000 -R /home/alpha/my-pak/torch-pak/
```

```bash
podman container run -it -v /home/alpha/my-pak/torch-pak:/home/app/project:Z -u 1000 pypak
```

```bash
docker container run -it -v /home/alpha/my-pak/torch-pak:/home/app/project:Z -u 1000 pypak
```

```bash
ls -al project/
sudo chown alpha:alpha -R torch-pak/
ls -al /home/alpha/my-pak/torch-pak/
```

```bash
podman container run -it -v /home/alpha/my-pak/torch-pak:/home/app/project:Z pypak
```

```bash
docker container run -it -v /home/alpha/my-pak/torch-pak:/home/app/project:Z pypak
```

```bash
podman-compose -f docker-compose-pypak.yml run publish
```

```bash
docker compose -f docker-compose-pypak.yml run publish
```

```bash
podman system prune
```

```bash
docker system prune
```


```bash
# podman play/generate/pod
```

```bash
podman generate kube eda197439e71 > torch-app-docker/pod-pypak.yaml
```

```bash
podman play kube torch-app-docker/pod-pypak.yaml
```

```bash
podman exec -it 75fbdb367eb7 /bin/bash
```

```bash
podman pod --help
podman pod ls
podman pod inspect 8902921088cb
podman pod top 8902921088cb 
```

```bash
podman unshare chown 1000:1000 -R /home/alpha/my-pak/torch-pak/
podman unshare ls -al /home/alpha/my-pak/torch-pak/
docker container run -it -v /home/alpha/my-pak/torch-pak:/home/app/project:Z -u 1000 pydev
```

>>>

```bash
ls -all
cd project/
cd torch-package-template/
emacs cookiecutter.json
cookiecutter torch-package-template/
ls torch-app-docker/
cd torch-app-docker/
pyproject-build
ls -all
```

```bash
podman unshare chown 1000:1000 -R /home/alpha/my-pak/torch-pak/

podman unshare ls -al /home/alpha/my-pak/torch-pak/

ls -al torch-pak/

sudo chown alpha:alpha -R torch-pak/

docker container run -it -v /home/alpha/my-pak/torch-pak:/home/app/project:Z -u 1000 pydev
```

