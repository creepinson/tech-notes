---
title: "Installing Kubernetes"
tags:
    - "Advanced"
    - "Kubernetes"
slug: "kube-setup"
description: "Setting up a k3s cluster with rancher"
draft: true
---

## Requirements

- A linux machine
- **kubectl**, [**kubectx**](https://github.com/ahmetb/kubectx) installed on your system

## Setting Up The Kubernetes Cluster

First we need to initialize the kubernetes cluster.

```bash
# Install k3s
curl -sLS https://get.k3sup.dev | sudo sh
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
# Setup kubernetes cluster
sudo k3sup install --local
sudo chown $USER $HOME/kubeconfig
export KUBECONFIG=$HOME/kubeconfig
# Check if node is ready
k3s kubectl get node
```

## Installing Rancher

```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
kubectl create namespace cattle-system
```

### Installing Cert Manager

```bash
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.crds.yaml
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.yaml
```

### Installing Rancher Using Helm

```bash
# Set the rancher domain
export RANCHER_HOST="rancher.theoparis.com"

helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set bootstrapPassword=toor \
  --set ingress.tls.source=letsEncrypt \
  --set hostname=$RANCHER_HOST
```

### Checking the Status Of Rancher

The rancher pods should all say either running or completed.

```bash
kubectl get pods -n cattle-system
```

## Using Rancher

You should now be able to visit rancher at the rancher host you configured,
with a bootstrap password of `toor`.

## Next Steps

[Now that you've got rancher setup,
you can create your first deployment.](/posts/kube-deploy)

