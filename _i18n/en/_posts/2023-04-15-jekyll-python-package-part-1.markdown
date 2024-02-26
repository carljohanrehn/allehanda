---
layout: post
title: Cross Python Development with Rootless Podman Part 2
date: 2023-04-15 21:32:08
description: 
categories: 
  - Cross Python Development
tags: 
  - Python
  - Jekyll
  - Podman
---

This post continues where we left off in [Cross Python Development with Rootless Podman Part 1](https://carljohanrehn.github.io/allehanda/blog/2023/python-package-part-2/). Here you'll see how to create a Python virtual environment for a specific project of yours, and how to activate it.

Always set a working directory

```dockerfile
FROM pydev-jekyll
USER app
WORKDIR /home/app
```

Create virtual environment (and delete existing if it exists) using a `requirements.txt` file

```dockerfile
COPY requirements.txt .

RUN py -3.10 -m venv /home/app/virtualenv/.venv && \
    /home/app/virtualenv/.venv/bin/pip install --upgrade pip && \
    /home/app/virtualenv/.venv/bin/pip install -r requirements.txt && \
    rm requirements.txt
```

For instance, my `requirements.txt` file contains the following

```
seaborn>=0.12.2
matplotlib>=3.6.2
numpy>=1.23.5
tqdm>=4.64
pandas>=1.5.2
scikit-learn
ipython
jupyter
jupyterlab
```

Add PyTorch (GPU version; for CPU version, see below)

```
torch
torchvision
torchaudio
```

Activate your virtual environment

```dockerfile
RUN source /home/app/virtualenv/.venv/bin/activate
```

Install with source package (*.tar.gz), Python version optional with tar.gz package

```dockerfile
RUN py -3.10 -m venv /home/app/torch-app/.venv && \
    /home/app/torch-app/.venv/bin/pip install --upgrade pip && \
    /home/app/torch-app/.venv/bin/pip install /home/app/project/dist/torch-app-0.0.1.tar.gz
```

or wheel (*.whl)

```dockerfile
RUN py -3.10 -m venv /home/app/torch-app/.venv && \
    /home/app/torch-app/.venv/bin/pip install --upgrade pip && \
   /home/app/torch-app/.venv/bin/pip install /home/app/project/dist/torch_app-0.0.1-non-any.whl
```

Install PyTorch CPU version with source or binary package 

```dockerfile
RUN py -m pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cpu
```

Jupyter notebook (or lab); set entrypoint in (alternatively, use docker compose file)

```dockerfile
CMD py -m notebook --no-browser --allow-root --ip=0.0.0.0 --port=8888 --notebook-dir=/home/app --NotebookApp.token='' --NotebookApp.password=''
# CMD py -m jupyter lab --no-browser --ip=0.0.0.0 --port=8888 --notebook-dir=/home/app --NotebookApp.token='' --NotebookApp.password=''
```

Optional: add accompanying code (from book "Inside Deep Learning")

```dockerfile
RUN git clone https://github.com/EdwardRaff/Inside-Deep-Learning.git && \
    mv Inside-Deep-Learning references/inside-deep-learning
```

Clone other books here...


```bash
podman image build --no-cache --format docker -f Containerfile-pypak-jekyll -t pypak-jekyll .
podman image build --format docker -f Containerfile-pypak-jekyll -t pypak-jekyll .
```

```bash
podman container run -it pypak-jekyll
podman container run -it --rm pypak-jekyll
```

```bash
# https://github.com/containers/podman/issues/3683
# https://unix.stackexchange.com/questions/728801/host-wide-consequences-of-setting-selinux-z-z-option-on-container-bind-mounts
# https://manpages.ubuntu.com/manpages/jammy/man1/docker-container-create.1.html
```

```bash
podman container run -it --rm -v .:/home/app/torch-app:Z pypak-jekyll
```

```bash
docker image build -f Containerfile-pypak-jekyll -t pypak-jekyll .
```

```bash
docker container run -it pypak-jekyll
docker container run -it --rm pypak-jekyll
```

```bash
# Jupyter notebook port mapping
docker container run -it -p 8888:8888 pypak-jekyll
```
