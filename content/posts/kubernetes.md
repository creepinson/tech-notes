---
title: "Installing Kubernetes"
tags:
    - "Advanced"
    - "Kubernetes"
slug: "kube-setup"
description: "Setting up a k3s cluster with rancher"
---

## Prequisites

- A linux machine
- **kubectl** & **helm**

## Setting Up The Kubernetes Cluster

First we need to initialize the kubernetes cluster.

```bash
# Install k3s
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens

# Setup kubernetes cluster
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--no-deploy traefik" sh -s -
sudo ln -s /etc/rancher/k3s/k3s.yaml ~/kubeconfig
sudo chown $USER $HOME/kubeconfig
export KUBECONFIG=$HOME/kubeconfig
# Check if node is ready
kubectl get node
```

## Next Steps

[Now that you've got k3s setup,
you can create your first deployment.](/posts/kube-deploy)

