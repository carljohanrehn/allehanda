---
layout: post
title:  Rootless Podman with Python and Jekyll
date: 2023-04-23 13:59:42
description: 
categories: 
  - Cross Python Development
tags: 
  - Python
  - Jekyll
  - Podman

---

In this post I'll describe a Podman [Containerfile](https://docs.podman.io/en/stable/markdown/podman-build.1.html) which describes how to set up a Python development environment with rootless Podman. It is based on a [gist](https://gist.github.com/BrutalSimplicity/882af1d343b7530fc7e005284523d38d) published by "BrutalSimplicity" and Dane Hillard's book [Publishing Python Packages](https://www.manning.com/books/publishing-python-packages). Furthermore, it also contains useful development tools and a full Ruby installation (necessary for creating [Jekyll](https://jekyllrb.com/) static webpages and blog posts, and [GitHub Pages](https://pages.github.com/)). Thus,

```dockerfile
FROM debian:bullseye

# USER root

LABEL maintainer="Carl Johan Rehn <care02@gmail.com>"
LABEL updated_at=2023-03-06

RUN apt-get update -y
```

Add locale and time zone (check in running container with $TZ, date, and locale), see question on  [stackoverflow](https://stackoverflow.com/questions/28405902/how-to-set-the-locale-inside-a-debian-ubuntu-docker-container) and Jiménez's [Gist on GitHub](https://gist.github.com/sjimenez44/1b73afeae3eec26a1915b0d4d5873b8f)

```dockerfile
RUN apt-get install -y locales locales-all tzdata

RUN sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen && \
    locale-gen
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8
# ENV TZ="America/New_York"
```

Local settings

```dockerfile
# RUN sed -i '/sv_SE.UTF-8/s/^# //g' /etc/locale.gen && \
#     locale-gen
# ENV LANG sv_SE.UTF-8
# ENV LANGUAGE sv_SE:sv
# ENV LC_ALL sv_SE.UTF-8
ENV TZ="Europe/Stockholm"
```

```dockerfile
ARG DEBIAN_FRONTEND=noninteractive
```

Basic development packages, see BrutalSimplicity's [Gist on GitHub](https://gist.github.com/BrutalSimplicity/882af1d343b7530fc7e005284523d38d)

```dockerfile
RUN apt-get clean && apt-get update && apt-get -y install --no-install-recommends \
    apt-utils \
    openssh-client \
    git \
    gnupg2 \
    dirmngr \
    iproute2 \
    procps \
    lsof \
    htop \
    net-tools \
    psmisc \
    curl \
    wget \
    rsync \
    ca-certificates \
    unzip \
    zip \
    nano \
    vim \
    neovim \
    emacs-nox \
    less \
    jq \
    lsb-release \
    apt-transport-https \
    dialog \
    libc6 \
    libgcc1 \
    libgcc1 \
    libkrb5-3 \
    libgssapi-krb5-2 \
    libicu[0-9][0-9] \
    libstdc++6 \
    zlib1g \
    locales \
    sudo \
    ncdu \
    man-db \
    strace \
    manpages \
    manpages-dev \
    init-system-helpers \
    make \
    build-essential \
```

Python runtime dependencies

```dockerfile
    openssl \
    bzip2 \
    libreadline8 \
    sqlite3 \
    tk \
    xz-utils \
    libxml2 \
    llvm \
```

Python build dependencies

```dockerfile
    build-essential \
    libssl-dev \
    zlib1g-dev \
    libbz2-dev \
    libreadline-dev \
    libsqlite3-dev \
    libncursesw5-dev \
    tk-dev \
    libxml2-dev \
    libxmlsec1-dev \
    libffi-dev \
    liblzma-dev \
```

Some useful development utilities

```dockerfile
    fd-find \
    bat \
    tree \
    zsh \
```

Additional tools

```dockerfile
    pandoc \
    imagemagick && \
    ln -s $(which fdfind) /usr/local/bin/fd && \
    ln -s $(which batcat) /usr/local/bin/bat && \
    rm -rf /var/lib/apt/lists/*
```

Add ruby (required for jekyll)

```dockerfile
RUN apt-get update
RUN apt-get upgrade
RUN apt-get install --no-install-recommends ruby-full build-essential zlib1g-dev -y
RUN apt-get install imagemagick -y
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/
```

Use bash for all RUN commands 

```dockerfile
SHELL ["/bin/bash", "-lc"]
```

Py launcher (requires root)

```dockerfile
RUN curl --location --remote-name https://github.com/brettcannon/python-launcher/releases/download/v1.0.0/python_launcher-1.0.0-x86_64-unknown-linux-gnu.tar.xz && \
    tar --extract --strip-components 1 --directory /usr/local --file python_launcher-1.0.0-x86_64-unknown-linux-gnu.tar.xz && \
    rm python_launcher-1.0.0-x86_64-unknown-linux-gnu.tar.xz
```

For hints on how to set up a rootless Podman image, see [Python Speed](https://pythonspeed.com/articles/root-capabilities-docker-security/) and
questions on [stackoverflow](https://stackoverflow.com/questions/59840450/rootless-docker-image). Create a non-root user

```dockerfile
RUN useradd --create-home app

USER app
```

Always set a working directory

```dockerfile
WORKDIR /home/app
```

Install [asdf](https://github.com/asdf-vm/asdf), [The Multiple Runtime Version Manager](https://asdf-vm.com/) and Python plugin

```dockerfile
RUN git clone https://github.com/asdf-vm/asdf.git $HOME/.asdf --branch v0.11.1 && \
    echo ". $HOME/.asdf/asdf.sh" >> $HOME/.bashrc && \
    echo ". $HOME/.asdf/asdf.sh" >> $HOME/.profile

RUN asdf plugin add python
    
# install python (asdf builds from source)
RUN asdf install python 3.10.8 

# set global versions
RUN asdf global python 3.10.8
```

Upgrade python package manager

```dockerfile
RUN pip install --upgrade pip
```

Install [pipx](https://github.com/pypa/pipx/)

```dockerfile
RUN py -3.10 -m pip install pipx && \
    asdf reshim python && \
    pipx ensurepath
```

Install [build](https://github.com/pypa/build), [tox](https://tox.wiki/en/latest/), [pre-commit](https://pre-commit.com/), and [cookiecutter](https://cookiecutter.readthedocs.io/)

```dockerfile
RUN pipx install build && \
    pipx install tox && \
    pipx install pre-commit && \
    pipx install cookiecutter
```

Optional: add accompanying code from book [Publishing Python Packages](https://www.manning.com/books/publishing-python-packages)

```dockerfile
RUN mkdir references && \
    git clone https://github.com/daneah/publishing-python-packages.git && \
    mv publishing-python-packages references/publishing-python-packages
```

Let's build an image with podman (remember to use use the [format](https://github.com/containers/podman/issues/8477) docker flag; the [no-cache](https://docs.podman.io/en/latest/markdown/podman-build.1.html) flag is optional). Hence, 

```bash
podman image build --no-cache --format docker -f Containerfile-pydev-jekyll -t pydev-jekyll .
```

Now, create a container

```bash
podman container run -it pydev-jekyll
```

and check that Python is available

```bash
app@8a709f7db41a:~$ py
Python 3.10.8 (main, Apr 11 2023, 11:43:31) [GCC 10.2.1 20210110] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.executable
'/home/app/.asdf/installs/python/3.10.8/bin/python3.10'
```

