---
layout: post
title:  Podman Local Registry
date: 2023-04-24 10:13:48
description: 
tags: Podman local-registry
categories: cross-python-development
---

---

- Location: ~/my-pak/cross-python-development/
- File: README-podman-local-registry

---

This post describes how to set up a local registry for Podman (and K3S). It is based on the following references

- https://blog.while-true-do.io/podman-local-container-registry/
- https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md
- https://blog.nashcom.de/nashcomblog.nsf/dx/k3s-podman-and-a-registry.htm

Copy registries.conf from /usr/share/containers or /etc/containers and modify it

```bash
emacs HOME/.config/containers/registries.conf
```

```bash
# ...

# Local registry...
[[registry]]
location = "localhost:5000"
insecure = true

# ...
```

The easiest way to run a local registry with Podman is to use the official docker registry container

```bash
podman container run -dt -p 5000:5000 --name registry docker.io/library/registry:2
```

Adding persistent storage can be done either with a named volume or a desired path

```bash
podman container run -dt -p 5000:5000 --name registry --volume registry:/var/lib/registry:Z docker.io/library/registry:2
```

If you want to see what's in your local registry, use

```bash
podman image search localhost:5000/ --tls-verify=false
```

Before pushing images to the local registry, you need to tag them accordingly

```bash
podman image tag localhost/pydev:latest localhost:5000/pydev:latest
```

```bash
podman image tag localhost/pypak:latest localhost:5000/pypak:latest
```

After that's done, it's an easy task to push your images to the registry

```bash
podman image push localhost:5000/pydev:latest --tls-verify=false
```

```bash
podman image push localhost:5000/pypak:latest --tls-verify=false
```

Let's see if we can find your images in the registry

```bash
podman image search localhost:5000/ --tls-verify=false
```

Pulling your images from your local Podman registry is easy

```bash
podman image pull localhost:5000/pydev:latest --tls-verify=false
```

```bash
podman image pull localhost:5000/pypak:latest --tls-verify=false
```
