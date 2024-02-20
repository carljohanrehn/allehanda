---
layout: post
title:  Jekyll Podman Rootless Part 1
date: 2023-04-15 11:31:43
description: 
tags: Jekyll, Podman
categories: fries-al-folio
---

---

- Location: ~/my-pak/cross-python-development/torch-app-docker/jekyll/docker/fries-al-folio/
- File: README-jekyll-podman-rootless

---

Suppose you want to run a Podman rootless container with Jekyll. The following container file shows how to create an image with Jekyll. Open it with your favourite editor (Emacs)

```bash
emacs Containerfile-pypak-jekyll-srv
```

You can build the image with Podman's build command

```bash
podman image build --no-cache --format docker -f Containerfile-pypak-jekyll-srv -t pypak-jekyll-srv .
```

Before running the rootless container, you need to ensure that root is not the owner of your project files. Let's see what we have on our host

```bash
ls -al
```

```bash
drwxr-xr-x. 1 alpha alpha  1700 Apr 29 09:34  .
drwxr-xr-x. 1 alpha alpha   190 Mar 18 10:38  ..
-rw-r--r--. 1 alpha alpha   243 Feb  7 17:08  404.html
-rw-r--r--. 1 alpha alpha  1390 Feb  7 17:08  .all-contributorsrc
drwxr-xr-x. 1 alpha alpha    58 Feb  7 17:08  assets
drwxr-xr-x. 1 alpha alpha    20 Feb  7 17:08  _bibliography
drwxr-xr-x. 1 alpha alpha   126 Feb  7 17:08  bin
drwxr-xr-x. 1 alpha alpha    20 Feb  7 17:08  blog
```

Clearly, it's your non-host user (alpha) that owns the files.

Now, using Podman's unshare command, we can see who the owner of the files in the running container is

```bash
podman unshare ls -al
```

```bash
drwxr-xr-x. 1 root root  1700 Apr 29 09:34  .
drwxr-xr-x. 1 root root   190 Mar 18 10:38  ..
-rw-r--r--. 1 root root   243 Feb  7 17:08  404.html
-rw-r--r--. 1 root root  1390 Feb  7 17:08  .all-contributorsrc
drwxr-xr-x. 1 root root    58 Feb  7 17:08  assets
drwxr-xr-x. 1 root root    20 Feb  7 17:08  _bibliography
drwxr-xr-x. 1 root root   126 Feb  7 17:08  bin
drwxr-xr-x. 1 root root    20 Feb  7 17:08  blog
```

Thus, your non-root user on the host corresponds to root! This is not what we wish for, so we need to change (or map) the owner to "app". 

Map root (uid 0) in the container to your non-root user on the host. This way when a container runs as root, it’s actually running as you!

```bash
# https://blog.christophersmart.com/2021/01/26/user-ids-and-rootless-containers-with-podman/
```

```bash
id -u
```

```bash
1000
```

Confirm the user range is applied for our user

```bash
podman unshare cat /proc/self/uid_map
```

```bash
0       1000          1
1     100000      65536
```

The root account with uid 0 in a container actually maps to the 1000 uid of your non-root user on the host. Then, the uid of 1 in a container for my user would map to 100000 on the host, 2 would be 100001 and so on. Thus, running as user "app" in a rootless container with uid of 1000 would map to 100999 on the host for alpha.

Now, let's set the owner to 1000, that is, the user "app", in the container,

```bash
podman unshare chown 1000:1000 -R .
```

and check the owner again,

```bash
ls -al
```

```bash
drwxr-xr-x. 1 100999 100999  1518 Apr 29 10:04 .
drwxr-xr-x. 1 alpha  alpha     44 Mar 25 08:08 ..
-rw-r--r--. 1 100999 100999   243 Feb  7 17:08 404.html
-rw-r--r--. 1 100999 100999  1390 Feb  7 17:08 .all-contributorsrc
drwxr-xr-x. 1 100999 100999    58 Feb  7 17:08 assets
drwxr-xr-x. 1 100999 100999    20 Feb  7 17:08 _bibliography
drwxr-xr-x. 1 100999 100999   126 Feb  7 17:08 bin
drwxr-xr-x. 1 100999 100999    20 Feb  7 17:08 blog
```

```bash
podman unshare ls -al
```

```bash
drwxr-xr-x. 1 alpha alpha  1518 Apr 29 10:04 .
drwxr-xr-x. 1 root  root     44 Mar 25 08:08 ..
-rw-r--r--. 1 alpha alpha   243 Feb  7 17:08 404.html
-rw-r--r--. 1 alpha alpha  1390 Feb  7 17:08 .all-contributorsrc
drwxr-xr-x. 1 alpha alpha    58 Feb  7 17:08 assets
drwxr-xr-x. 1 alpha alpha    20 Feb  7 17:08 _bibliography
drwxr-xr-x. 1 alpha alpha   126 Feb  7 17:08 bin
drwxr-xr-x. 1 alpha alpha    20 Feb  7 17:08 blog
```

It's time to create a rootless container. You can do that by running a service defined in a docker compose file. Open it and take notice of how the jekyll service is defined,

```bash
emacs docker-compose-project-podman.yml
```

Run the service with podman-compose,

```bash
podman-compose -f docker-compose-project-podman.yml up
```

You can shut down the service with the "down" command

```bash
podman-compose -f docker-compose-project-podman.yml down
```

Finally, revert back and set the local owner to alpha and check that you're back where you started

```bash
sudo chown alpha:alpha -R .
```

```bash
ls -al
```

```bash
drwxr-xr-x. 1 alpha alpha  1700 Apr 29 09:34  .
drwxr-xr-x. 1 alpha alpha   190 Mar 18 10:38  ..
-rw-r--r--. 1 alpha alpha   243 Feb  7 17:08  404.html
-rw-r--r--. 1 alpha alpha  1390 Feb  7 17:08  .all-contributorsrc
drwxr-xr-x. 1 alpha alpha    58 Feb  7 17:08  assets
drwxr-xr-x. 1 alpha alpha    20 Feb  7 17:08  _bibliography
drwxr-xr-x. 1 alpha alpha   126 Feb  7 17:08  bin
drwxr-xr-x. 1 alpha alpha    20 Feb  7 17:08  blog
```

and who the owner will be in the container using Podman's unshare command

```bash
podman unshare ls -al
```

```bash
drwxr-xr-x. 1 root root  1700 Apr 29 09:34  .
drwxr-xr-x. 1 root root   190 Mar 18 10:38  ..
-rw-r--r--. 1 root root   243 Feb  7 17:08  404.html
-rw-r--r--. 1 root root  1390 Feb  7 17:08  .all-contributorsrc
drwxr-xr-x. 1 root root    58 Feb  7 17:08  assets
drwxr-xr-x. 1 root root    20 Feb  7 17:08  _bibliography
drwxr-xr-x. 1 root root   126 Feb  7 17:08  bin
drwxr-xr-x. 1 root root    20 Feb  7 17:08  blog
```



