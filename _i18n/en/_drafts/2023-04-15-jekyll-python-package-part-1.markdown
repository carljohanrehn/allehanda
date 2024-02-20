---
layout: post
title:  Jekyll Python Package Part 1
date: 2023-04-15 21:32:08
description: 
tags: Jekyll, Podman
categories: fries-al-folio
---

---

- Location: ~/my-pak/cross-python-development/torch-app-docker/
- File: Containerfile-pypak-jekyll

---


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


```dockerfile
FROM pydev-jekyll

USER app

# always set a working directory
WORKDIR /home/app

# create virtual environment (and delete existing if it exists)
COPY requirements.txt .

RUN py -3.10 -m venv /home/app/virtualenv/.venv && \
    /home/app/virtualenv/.venv/bin/pip install --upgrade pip && \
    /home/app/virtualenv/.venv/bin/pip install -r requirements.txt && \
    rm requirements.txt

# activate virtual enviroment
# https://www.freecodecamp.org/news/manage-multiple-python-versions-and-virtual-environments-venv-pyenv-pyvenv-a29fb00c296f/
# RUN source /home/app/venv/.venv/bin/activate
# RUN source /home/app/torch-app/.venv/bin/activate

# install with source package (*.tar.gz) or wheel (*.whl)
# Python version optional with tar.gz package; otherwise, needs to be specified when using wheel
# RUN py -3.10 -m venv /home/app/torch-app/.venv && \
#     /home/app/torch-app/.venv/bin/pip install --upgrade pip && \
#     /home/app/torch-app/.venv/bin/pip install /home/app/project/dist/torch-app-0.0.1.tar.gz
##     /home/app/torch-app/.venv/bin/pip install /home/app/project/dist/torch_app-0.0.1-non-any.whl

# special for PyTorch CPU version with source or binary package installation, see README.md
# RUN py -m pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cpu

# accompanying code (from book "Inside Deep Learning")
RUN git clone https://github.com/EdwardRaff/Inside-Deep-Learning.git && \
    mv Inside-Deep-Learning references/inside-deep-learning

# clone other books here...

# jupyter notebook (or lab); set entrypoint in docker compose file...
# CMD py -m notebook --no-browser --allow-root --ip=0.0.0.0 --port=8888 --notebook-dir=/home/app --NotebookApp.token='' --NotebookApp.password=''
# CMD py -m jupyter lab --no-browser --ip=0.0.0.0 --port=8888 --notebook-dir=/home/app --NotebookApp.token='' --NotebookApp.password=''
```
