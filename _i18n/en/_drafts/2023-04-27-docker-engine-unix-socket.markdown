---
layout: post
title:  Docker Engine Unix Socket
DATE: 2023-04-27 08:34:53
description: 
tags: docker-engine unix-socket
categories: cross-python-development
---

---

- Location: ~/my-pak/cross-python-development/
- File: README-docker-sock

---

```bash
# Set "TCP socket" in PyCharm Settings > Docker
# Use URL to Docker Desktop's Docker Engine unix socket (see docker context ls)
# https://youtrack.jetbrains.com/issue/RIDER-77609
```

```bash
unix:///home/alpha/.docker/desktop/docker.sock
```

```bash
# I got it working! Choose "TCP socket" then paste in the URL to Docker Desktop's Docker Engine unix socket (see docker context ls), and it just works!
```

```bash
# https://youtrack.jetbrains.com/issue/IDEA-294871
```

```bash
sudo ln -s /home/${user}/.docker/desktop/docker.sock /var/run/docker.sock
```

```bash
# In order to get docker desktop's compose to work you have to go into
# Settings >> Build, Execution, Deployment >> Tools
```

