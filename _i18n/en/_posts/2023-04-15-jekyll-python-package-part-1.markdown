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
--extra-index-url https://download.pytorch.org/whl/cpu
# --extra-index-url https://files.pythonhosted.org/packages/86/cc/f2685fc3fc37122fe8be22e6c0dfdaeab49026625b8c2cf41bc87b1cdd4d
# https://files.pythonhosted.org/packages/80/5e/f095ccdf24860a7548b39f93d2df03017ad3218f90a0406feb5e5661d0c7/scikit_learn-1.2.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
# https://files.pythonhosted.org/packages/80/53/47bed2190d2d86914223454f69744553521bc20c7de922d6a3b5300316ab/scikit_learn-1.2.1-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
seaborn>=0.12.2
matplotlib>=3.6.2
numpy>=1.23.5
tqdm>=4.64
pandas>=1.5.2
# sklearn
scikit-learn
torch
torchvision
torchaudio
# torch-cpu
# torchvision-cpu
# torchaudio-cpu
ipython
jupyter
jupyterlab
```

Activate your virtual environment

```dockerfile
RUN source /home/app/virtualenv/.venv/bin/activate
```

Install with source package (*.tar.gz) or wheel (*.whl). Python version optional with tar.gz package; otherwise, needs to be specified when using wheel

```dockerfile
RUN py -3.10 -m venv /home/app/torch-app/.venv && \
    /home/app/torch-app/.venv/bin/pip install --upgrade pip && \
    /home/app/torch-app/.venv/bin/pip install /home/app/project/dist/torch-app-0.0.1.tar.gz
#     /home/app/torch-app/.venv/bin/pip install /home/app/project/dist/torch_app-0.0.1-non-any.whl
```

Special for PyTorch CPU version with source or binary package installation, see README.md

```dockerfile
RUN py -m pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cpu
```

Optional: add accompanying code (from book "Inside Deep Learning")

```dockerfile
RUN git clone https://github.com/EdwardRaff/Inside-Deep-Learning.git && \
    mv Inside-Deep-Learning references/inside-deep-learning
```

Clone other books here...

Jupyter notebook (or lab); set entrypoint in docker compose file

```dockerfile
CMD py -m notebook --no-browser --allow-root --ip=0.0.0.0 --port=8888 --notebook-dir=/home/app --NotebookApp.token='' --NotebookApp.password=''
# CMD py -m jupyter lab --no-browser --ip=0.0.0.0 --port=8888 --notebook-dir=/home/app --NotebookApp.token='' --NotebookApp.password=''
```


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
