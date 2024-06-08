# Perpetulab

---

This is my perpetual homelab repo. 
The idea is for it to be easily deployable without any preconceived knowledge.
This includes any required tools and configurations.

---

## Installation

### Environment setup

Install all required tools:
```shell
brew bundle --no-lock
```

Create age secret for sops:
```shell
mkdir "$HOME/Library/Application Support/sops"
mkdir "$HOME/Library/Application Support/sops/age"
age-keygen -o "$HOME/Library/Application Support/sops/age/keys.txt"
```

Copy generated public key to `.sops.yaml`.

Save private key somewhere safe:
```shell
cat "$HOME/Library/Application Support/sops/age/keys.txt"
```

---

### Talos setup

#### Step 1 - write config

Create talsecret:
```shell
talhelper gensecret > talsecret.sops.yaml
```

Encrypt talsecret:
```shell
sops -e -i talsecret.sops.yaml
```

Add talsecret to git:
```shell
git add talsecret.sops.yaml
```

#### Step 2 - apply config

Generate config:
```shell
talhelper genconfig
```

Apply config:
```shell
talosctl apply -n 192.168.1.4 -f clusterconfig/perpetulab-cluster-homeserver.yaml -i
```

Bootstrap kubernetes:
```shell
talosctl bootstrap
```

Add to kubeconfig:
```shell
talosctl kubeconfig
```

#### Step 2.5 - add networking to finish cluster setup

Create observability namespace:
```shell
kubectl apply -f kubernetes/observability/observability-namespace.yaml
```

Install prometheus-operator-crds:
```shell
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
```shell
helm install prometheus-operator-crds prometheus-community/prometheus-operator-crds --namespace observability
```

Install cilium:
```shell
helm repo add cilium https://helm.cilium.io/
helm repo update
```
```shell
helm install cilium cilium/cilium -f kubernetes/infra/controller/kube-system/cilium/cilium-values.yaml --namespace kube-system
```

Add ip pools:
```shell
kubectl apply -f kubernetes/kube-system/cilium/external-ip-pool.yaml
kubectl apply -f kubernetes/kube-system/cilium/internal-ip-pool.yaml
```

Create traefik namespace:
```shell
kubectl apply -f kubernetes/traefik/namespace.yaml
```

Install traefik:
```shell
helm install -n=traefik traefik traefik/traefik -f kubernetes/traefik/values.yaml
```

#### Step 3 - setup flux

Create GitHub deploy key:

```shell
mkdir keys
ssh-keygen -t ecdsa -b 521 -C "github-deploy-key" -f keys/github-deploy.key -q -P ""
```

Add `keys/github-deploy.key.pub` to repository deploy keys (read only).

Bootstrap flux:
```shell
flux bootstrap git \
  --url=ssh://git@github.com/Marcin795/perpetulab.git \
  --branch=master \
  --private-key-file=keys/github-deploy.key \
  --path=kubernetes 
```
