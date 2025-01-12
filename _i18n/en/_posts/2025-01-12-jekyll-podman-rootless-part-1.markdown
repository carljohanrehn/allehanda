---
layout: post
title:  "Running Jekyll in a Rootless Podman Container: Part 1 - Setting the Stage"
date: 2023-04-15 11:31:43
description: "A guide on setting up and running Jekyll in a rootless Podman container. We explore the importance of user mapping and file ownership when using rootless containers."
tags: Jekyll, Podman, Containers, Rootless, Development
categories: fries-al-folio
---

---

In this first part of a two part series, we'll explore how to run Jekyll, a static site generator, inside a rootless Podman container. This approach enhances security and allows for more streamlined development workflows. We'll cover user mapping, file ownership, and how to run Jekyll with `podman-compose`.

This post will guide you through these steps, making use of the following folder:
```text
~/my-pak/cross-python-development/torch-app-docker/jekyll/docker/fries-al-folio/
```

The file to examine is named: `README-jekyll-podman-rootless`.

## Building the Jekyll Image

First, we need to create a container image that includes Jekyll. The image itself is built from the following `Containerfile` : `Containerfile-pypak-jekyll-srv`. You can open it with your favorite editor:

```bash
emacs Containerfile-pypak-jekyll-srv
```

Once you have examined it, we can build the image with Podman using the `build` command:

```bash
podman image build --no-cache --format docker -f Containerfile-pypak-jekyll-srv -t pypak-jekyll-srv .
```

This command will create a Docker-formatted image named `pypak-jekyll-srv` that we will use later.

## Understanding File Ownership in Rootless Containers

Before we can run a rootless container, we must address an important detail: file ownership. Let's see who owns the files on our host by running:
```bash
ls -al
```
You should see something like this:

```text
drwxr-xr-x. 1 alpha alpha  1700 Apr 29 09:34  .
drwxr-xr-x. 1 alpha alpha   190 Mar 18 10:38  ..
-rw-r--r--. 1 alpha alpha   243 Feb  7 17:08  404.html
-rw-r--r--. 1 alpha alpha  1390 Feb  7 17:08  .all-contributorsrc
drwxr-xr-x. 1 alpha alpha    58 Feb  7 17:08  assets
drwxr-xr-x. 1 alpha alpha    20 Feb  7 17:08  _bibliography
drwxr-xr-x. 1 alpha alpha   126 Feb  7 17:08  bin
drwxr-xr-x. 1 alpha alpha    20 Feb  7 17:08  blog
```
As expected, our user, *alpha*, owns these files.

Now, let's use `podman unshare` to see who owns the files inside the container when we run it.

```bash
podman unshare ls -al
```

The output will look like this:

```text
drwxr-xr-x. 1 root root  1700 Apr 29 09:34  .
drwxr-xr-x. 1 root root   190 Mar 18 10:38  ..
-rw-r--r--. 1 root root   243 Feb  7 17:08  404.html
-rw-r--r--. 1 root root  1390 Feb  7 17:08  .all-contributorsrc
drwxr-xr-x. 1 root root    58 Feb  7 17:08  assets
drwxr-xr-x. 1 root root    20 Feb  7 17:08  _bibliography
drwxr-xr-x. 1 root root   126 Feb  7 17:08  bin
drwxr-xr-x. 1 root root    20 Feb  7 17:08  blog
```

Notice that inside the container, the owner of the files is `root` (`uid=0`). This is because in a rootless container environment, user IDs from the container are mapped to user IDs on your host machine. This is not ideal!

## Mapping User IDs in Rootless Containers

We need to map the root user inside the container to our non-root user (`alpha`) on the host. This means that when a process inside the container runs as root, it's actually running as the user *alpha* on the host.

First, let's find our user's ID on the host:

```bash
id -u
```
It should return `1000`.

Next, check the user range applied for your user with the following:
```bash
podman unshare cat /proc/self/uid_map
```

The output should look similar to this:
```text
0       1000          1
1     100000      65536
```

This tells us the user ID mapping:

*   `uid 0` inside the container maps to `uid 1000` on the host (our non-root user).
*   `uid 1` inside the container maps to `uid 100000` on the host, and so on.

To use a different user inside our container, with an arbitrary uid, let's say 1000, we need to map this uid inside the container to the range of allowed uids in our host. Thus, we need to set the owner of the files inside the container to `1000:1000` which will be mapped to an allowed user in our host.
```bash
podman unshare chown 1000:1000 -R .
```
Let's check what we have now, by running:

```bash
ls -al
```

```text
drwxr-xr-x. 1 100999 100999  1518 Apr 29 10:04 .
drwxr-xr-x. 1 alpha  alpha     44 Mar 25 08:08 ..
-rw-r--r--. 1 100999 100999   243 Feb  7 17:08 404.html
-rw-r--r--. 1 100999 100999  1390 Feb  7 17:08 .all-contributorsrc
drwxr-xr-x. 1 100999 100999    58 Feb  7 17:08 assets
drwxr-xr-x. 1 100999 100999    20 Feb  7 17:08 _bibliography
drwxr-xr-x. 1 100999 100999   126 Feb  7 17:08 bin
drwxr-xr-x. 1 100999 100999    20 Feb  7 17:08 blog
```

and by running this command in the unshare context

```bash
podman unshare ls -al
```

```text
drwxr-xr-x. 1 alpha alpha  1518 Apr 29 10:04 .
drwxr-xr-x. 1 root  root     44 Mar 25 08:08 ..
-rw-r--r--. 1 alpha alpha   243 Feb  7 17:08 404.html
-rw-r--r--. 1 alpha alpha  1390 Feb  7 17:08 .all-contributorsrc
drwxr-xr-x. 1 alpha alpha    58 Feb  7 17:08 assets
drwxr-xr-x. 1 alpha alpha    20 Feb  7 17:08 _bibliography
drwxr-xr-x. 1 alpha alpha   126 Feb  7 17:08 bin
drwxr-xr-x. 1 alpha alpha    20 Feb  7 17:08 blog
```

Notice how the owners are different depending on if the command is run under the unshare context or not.

## Running Jekyll with podman-compose

Now that we have set up the file ownership, we can run our Jekyll container. We will use `podman-compose`. First, let's open our `docker-compose` file to understand the definition of the jekyll service:

```bash
emacs docker-compose-project-podman.yml
```

Once you understand how the service is defined, let's run the container:

```bash
podman-compose -f docker-compose-project-podman.yml up
```
This command starts the Jekyll service defined in your compose file.

To shut down the service, use:

```bash
podman-compose -f docker-compose-project-podman.yml down
```
## Reverting File Ownership

After running your container, it's important to revert the file ownership back to your host user:

```bash
sudo chown alpha:alpha -R .
```
Check if everything is correct by running `ls -al`:
```bash
ls -al
```
```text
drwxr-xr-x. 1 alpha alpha  1700 Apr 29 09:34  .
drwxr-xr-x. 1 alpha alpha   190 Mar 18 10:38  ..
-rw-r--r--. 1 alpha alpha   243 Feb  7 17:08  404.html
-rw-r--r--. 1 alpha alpha  1390 Feb  7 17:08  .all-contributorsrc
drwxr-xr-x. 1 alpha alpha    58 Feb  7 17:08  assets
drwxr-xr-x. 1 alpha alpha    20 Feb  7 17:08  _bibliography
drwxr-xr-x. 1 alpha alpha   126 Feb  7 17:08  bin
drwxr-xr-x. 1 alpha alpha    20 Feb  7 17:08  blog
```
And finally, let's confirm that the user inside the container will be root, by running the command within the podman context:
```bash
podman unshare ls -al
```

```text
drwxr-xr-x. 1 root root  1700 Apr 29 09:34  .
drwxr-xr-x. 1 root root   190 Mar 18 10:38  ..
-rw-r--r--. 1 root root   243 Feb  7 17:08  404.html
-rw-r--r--. 1 root root  1390 Feb  7 17:08  .all-contributorsrc
drwxr-xr-x. 1 root root    58 Feb  7 17:08  assets
drwxr-xr-x. 1 root root    20 Feb  7 17:08  _bibliography
drwxr-xr-x. 1 root root   126 Feb  7 17:08  bin
drwxr-xr-x. 1 root root    20 Feb  7 17:08  blog
```
That's it for now. In the next part we will delve into how to run and test your blog with a rootless container in a reproducible way.

---
*This is Part 1 of a series on using rootless Podman containers with Jekyll.*


