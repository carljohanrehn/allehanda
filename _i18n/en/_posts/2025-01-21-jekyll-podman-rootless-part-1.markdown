---
layout: post
title: "Running Jekyll in a Rootless Podman Container: Part 1 - Setting the Stage"
date: 2025-01-20 09:42:43
description: "A guide on setting up and running Jekyll in a rootless Podman container. We explore the importance of user mapping and file ownership when using rootless containers."
tags: [Containers, Development, Jekyll, Podman, Rootless]
categories: [fries-al-folio]
---

In this first part of a three-part series, we’ll explore how to run Jekyll, a static site generator, inside a rootless Podman container. Rootless containers enhance security by eliminating the need for root privileges, while also providing a lightweight and streamlined development workflow. This guide focuses on key aspects such as user mapping, file ownership, and running Jekyll with `podman-compose`.

---

## Setting Up the Podman Environment

We’ll assume you’re working in a project folder such as the following:

```text
~/my-pak/cross-python-development/torch-app-docker/jekyll/docker/fries-al-folio/
```

You’ll want to familiarize yourself with the provided `README` file for this project, named:

```text
README-jekyll-podman-rootless
```

Let’s dive into creating a container image for Jekyll and preparing the environment.

---

## Building the Jekyll Image

First, we need to create a Podman container image for Jekyll. The build process utilizes a `Containerfile`, specifically:

```text
Containerfile-pypak-jekyll-srv
```

Open the file in your preferred editor to review its contents:

```bash
emacs Containerfile-pypak-jekyll-srv
```

Once reviewed, build the container image using the following command:

```bash
podman image build --no-cache --format docker -f Containerfile-pypak-jekyll-srv -t pypak-jekyll-srv .
```

This will create a Docker-format container image named `pypak-jekyll-srv`, which we’ll use to run our Jekyll service.

---

## Understanding File Ownership in Rootless Containers

Next, let’s address a vital aspect of rootless containers: file ownership. File ownership may differ inside the container compared to the host machine.

### Checking Current File Ownership

Run the following command to inspect ownership for your project files:

```bash
ls -al
```

You should see output similar to this:

```text
drwxr-xr-x. 1 alpha alpha  1700 Apr 29 09:34  .
drwxr-xr-x. 1 alpha alpha   190 Mar 18 10:38  ..
-rw-r--r--. 1 alpha alpha   243 Feb  7 17:08  404.html
drwxr-xr-x. 1 alpha alpha    58 Feb  7 17:08  assets
...
```

Here, the files are owned by the user `alpha` on your host. We now use `podman unshare` to examine ownership inside the container.

### Inspecting File Ownership Inside the Container

Inside the container, run:

```bash
podman unshare ls -al
```

This will output something like:

```text
drwxr-xr-x. 1 root root  1700 Apr 29 09:34  .
drwxr-xr-x. 1 root root   190 Mar 18 10:38  ..
-rw-r--r--. 1 root root   243 Feb  7 17:08  404.html
drwxr-xr-x. 1 root root    58 Feb  7 17:08  assets
...
```

Notice that the container considers `root` as the owner of these files. This discrepancy occurs due to user ID mappings between the container and host.

---

## Mapping User IDs in Rootless Containers

To solve this ownership challenge, we’ll map the container’s `root` user to our `alpha` user on the host machine.

### Step 1: Identify Your Host User ID

Run the following command to find your user’s ID:

```bash
id -u
```

You should see output like:

```text
1000
```

This ID corresponds to the user `alpha`.

### Step 2: Examine the User Range

Next, check the user ID mapping range for rootless containers:

```bash
podman unshare cat /proc/self/uid_map
```

The output should look like this:

```text
0       1000          1
1     100000      65536
```

This indicates the mappings:

- `uid 0` in the container maps to `uid 1000` (current non-root user on the host).
- The range starting at `uid 1` maps to `uid 100000` and higher.

### Step 3: Adjust File Ownership for Compatibility

We need to ensure that files in our project are owned by `uid 1000:1000` in the container, which matches the host. Use the following command:

```bash
podman unshare chown 1000:1000 -R .
```

Check the results using:

```bash
ls -al
```

You’ll see the new ownership reflected as:

```text
drwxr-xr-x. 1 100999 100999  1518 Apr 29 10:04 .
drwxr-xr-x. 1 alpha  alpha     44 Mar 25 08:08 ..
-rw-r--r--. 1 100999 100999   243 Feb  7 17:08 404.html
...
```

Inside the unshared context of `podman`, ownership will appear correctly mapped back to your host's user:

```bash
podman unshare ls -al
```

```text
drwxr-xr-x. 1 alpha alpha  1518 Apr 29 10:04 .
drwxr-xr-x. 1 root  root     44 Mar 25 08:08 ..
-rw-r--r--. 1 alpha alpha   243 Feb  7 17:08 404.html
...
```

---

## Running Jekyll with `podman-compose`

Finally, we’re ready to run the Jekyll container. This setup uses `podman-compose` to define and manage container services.

### Step 1: Check Your Compose Configuration

Open your `docker-compose` file for review:

```bash
emacs docker-compose-project-podman.yml
```

Ensure the Jekyll service is defined properly.

### Step 2: Start the Container

Run the following command to start the Jekyll service:

```bash
podman-compose -f docker-compose-project-podman.yml up
```

Once launched, your Jekyll service will be available as per the compose file’s configuration.

To stop the service:

```bash
podman-compose -f docker-compose-project-podman.yml down
```

---

## Reverting File Ownership After Running the Container

Before ending your work, revert the file ownership of the project back to your host user:

```bash
sudo chown alpha:alpha -R .
```

Double-check the ownership:

```bash
ls -al
```

This ensures consistent permissions when working outside the context of Podman. File ownership inside the container will remain usable.

---

## Conclusion

In this post, we’ve covered how to build a custom Jekyll image, manage user ID mapping in rootless containers, and set up `podman-compose` to simplify containerized Jekyll workflows. Understanding file ownership and user mapping is vital for working securely with rootless containers.

Stay tuned for **Part 2**, where we’ll focus on running and testing Jekyll inside the container in a reproducible way!

---
*This is Part 1 of a series on using rootless Podman containers with Jekyll.*
