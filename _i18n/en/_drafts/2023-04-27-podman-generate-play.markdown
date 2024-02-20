---
layout: post
title:  Podman Generate and Play
DATE: 2023-04-27 11:55:07
description: 
tags: podman generate play pod deployment
categories: cross-python-development
---

---

- Location: ~/my-pak/cross-python-development/
- Files: pod-pypak.yaml, deployment-pypak-dev

---

```bash
podman generate kube eda197439e71 > torch-app-docker/pod-pypak.yaml
podman play kube deployment-pypak-dev.yaml

podman-compose -f docker-compose-project-podman.yml up
```




```yaml
# Save the output of this file and use kubectl create -f to import
# it into Kubernetes.
#
# Created with podman-4.3.1

# NOTE: If you generated this yaml from an unprivileged and rootless podman container on an SELinux
# enabled system, check the podman generate kube man page for steps to follow to ensure that your pod/container
# has the right permissions to access the volumes added.
---
apiVersion: v1
kind: Pod
metadata:
  annotations:
    bind-mount-options: /home/alpha/my-pak/torch-pak/torch-app-docker:Z
    io.kubernetes.cri-o.TTY/torch-app-dockerpublishtmp51069: "true"
    io.podman.annotations.autoremove/torch-app-dockerpublishtmp51069: "FALSE"
    io.podman.annotations.init/torch-app-dockerpublishtmp51069: "FALSE"
    io.podman.annotations.privileged/torch-app-dockerpublishtmp51069: "FALSE"
    io.podman.annotations.publish-all/torch-app-dockerpublishtmp51069: "FALSE"
  creationTimestamp: "2023-03-14T08:32:36Z"
  labels:
    app: torch-app-dockerpublishtmp51069-pod
  name: torch-app-dockerpublishtmp51069-pod
spec:
  automountServiceAccountToken: false
  containers:
  - image: localhost:5000/pypak:latest
    name: torch-app-dockerpublishtmp51069
    securityContext:
      capabilities:
        drop:
        - CAP_MKNOD
        - CAP_NET_RAW
        - CAP_AUDIT_WRITE
    stdin: true
    tty: true
    volumeMounts:
    - mountPath: /home/app/project
      name: home-alpha-my-pak-torch-pak-torch-app-docker-host-0
  enableServiceLinks: false
  volumes:
  - hostPath:
      path: /home/alpha/my-pak/torch-pak/torch-app-docker
      type: Directory
    name: home-alpha-my-pak-torch-pak-torch-app-docker-host-0
```

```yaml
# Save the output of this file and use kubectl create -f to import
# it into Kubernetes.
#
# Created with podman-4.3.1

# NOTE: If you generated this yaml from an unprivileged and rootless podman container on an SELinux
# enabled system, check the podman generate kube man page for steps to follow to ensure that your pod/container
# has the right permissions to access the volumes added.
---
apiVersion: v1
kind: Pod
metadata:
  annotations:
    bind-mount-options: /home/alpha/my-pak/torch-pak/torch-app-docker:Z
    io.kubernetes.cri-o.TTY/torch-app-dockerpublishtmp51069: "true"
    io.podman.annotations.autoremove/torch-app-dockerpublishtmp51069: "FALSE"
    io.podman.annotations.init/torch-app-dockerpublishtmp51069: "FALSE"
    io.podman.annotations.privileged/torch-app-dockerpublishtmp51069: "FALSE"
    io.podman.annotations.publish-all/torch-app-dockerpublishtmp51069: "FALSE"
  creationTimestamp: "2023-03-14T08:32:36Z"
  labels:
    app: torch-app-dockerpublishtmp51069-pod
  name: torch-app-dockerpublishtmp51069-pod
spec:
  automountServiceAccountToken: false
  containers:
  - image: localhost:5000/pypak:latest
    name: torch-app-dockerpublishtmp51069
    securityContext:
      capabilities:
        drop:
        - CAP_MKNOD
        - CAP_NET_RAW
        - CAP_AUDIT_WRITE
    stdin: true
    tty: true
    volumeMounts:
    - mountPath: /home/app/project
      name: home-alpha-my-pak-torch-pak-torch-app-docker-host-0
  enableServiceLinks: false
  volumes:
  - hostPath:
      path: /home/alpha/my-pak/torch-pak/torch-app-docker
      type: Directory
    name: home-alpha-my-pak-torch-pak-torch-app-docker-host-0
```

