---
layout: post
title:  K3S Local Registry
DATE: 2023-04-27 08:25:54
description: 
tags: K3S local-registry
categories: cross-python-development
---

---

- Location: ~/my-pak/cross-python-development/
- File: README-k3s-local-registry

---

```bash
# Create k3s local registry
```

```bash
## https://blog.nashcom.de/nashcomblog.nsf/dx/k3s-podman-and-a-registry.htm
# https://docs.k3s.io/installation/private-registry
# https://livebook.manning.com/book/learn-kubernetes-in-a-month-of-lunches/chapter-2/103
```


```bash
emacs /etc/rancher/k3s/registries.yaml
```

```yaml
  mirrors:
    localhost:
      endpoint:
        - "http://localhost:5000"
```

```bash
systemctl restart k3s
```


```bash
kubectl apply -f pod-pypak.yaml
kubectl get pod
kubectl describe pod torch-app-dockerpublishtmp51069-pod
```

```bash
# kubectl get pods
# kubectl delete pod torch-app-dockerpublishtmp51069-pod
```

```bash
kubectl get pod torch-app-dockerpublishtmp51069-pod --output custom-columns=NAME:metadata.name,NODE_IP:status.hostIP,PO
D_IP:status.podIP
```

```bash
kubectl exec -it torch-app-dockerpublishtmp51069-pod -- /bin/bash
```

```bash
  ls -al
  source virtualenv/.venv/bin/activate
  py project/src/torchappdocker/torchapp/ch01.py
```

```bash
  py -m IPython
```

```python
    from sklearn.datasets import fetch_openml

    X, y = fetch_openml('mnist_784', version=1, return_X_y=True, as_frame=False, parser='auto')

    # TODO Fix URL access: "Temporary failure in name resolution..."
```

