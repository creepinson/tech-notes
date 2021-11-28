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
- Ssh keys on your servers (using `.ssh/id_ed25519`)

## Setting Up The Kubernetes Cluster

First we need to initialize the kubernetes cluster.

```bash
# Install k3s dependencies
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
curl -sLS https://get.k3sup.dev | sh
sudo mv k3sup /usr/local/bin/

# The main node of the cluster
export SERVER_IP=192.168.1.177

# Setup kubernetes cluster (requires ssh ed25519 keys to be setup)
# See https://github.com/alexellis/k3sup#-setup-a-kubernetes-server-with-k3sup
k3sup install --ip $SERVER_IP --user $USER --ssh-key ~/.ssh/id_ed25519
export KUBECONFIG=~/kubeconfig
kubectl config set-context default

# Check if node is ready
kubectl get node
```

## Joining a Second Node (Optional)

Make sure that your second node can reach your first node's ip address beforehand.

```zsh
# The new node to join into your cluster
export AGENT_IP="192.168.1.99"

k3sup join --ip $AGENT_IP --server-ip $SERVER_IP --user $USER --ssh-key ~/.ssh/id_ed25519
```

## Next Steps

[Now that you've got k3s setup,
you can create your first deployment.](/posts/kube-deploy)
[You can also monitor your cluster with k9s](https://github.com/derailed/k9s).

