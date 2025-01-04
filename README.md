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

Add helm repositories required for bootstrap:
```shell
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add cilium https://helm.cilium.io/
helm repo update
```

Create GitHub deploy key:

```shell
mkdir keys
ssh-keygen -t ecdsa -b 521 -C "github-deploy-key" -f keys/github-deploy.key -q -P ""
```

Add `keys/github-deploy.key.pub` to repository deploy keys (read only).

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

---

### Talos setup

#### Step 1 - generate config

Generate config:
```shell
talhelper genconfig --no-gitignore
```

#### Step 2 - apply config

Apply config:
```shell
talosctl apply -n 192.168.8.4 -f clusterconfig/perpetulab-cluster-homeserver.yaml -i
```

Bootstrap kubernetes:
```shell
talosctl bootstrap
```

Add to kubeconfig:
```shell
talosctl kubeconfig
```

#### Step 3 - add services required before flux

Create observability namespace:
```shell
kubectl apply -f infra/controllers/observability/namespace.yaml
```

Install prometheus-operator-crds:
```shell
helm install prometheus-operator-crds prometheus-community/prometheus-operator-crds --namespace observability
```

Install cilium:
```shell
helm install cilium cilium/cilium -f infra/controllers/network/cilium/cilium-values.yaml --namespace kube-system
```

Install csr approver:
```shell
helm install kubelet-csr-approver postfinance/kubelet-csr-approver -f infra/controllers/network/csr-approver/approver-values.yaml --namespace kube-system
```

#### Step 4 - setup flux

Bootstrap flux:
```shell
flux bootstrap git \
  --url=ssh://git@github.com/Marcin795/perpetulab.git \
  --branch=master \
  --private-key-file=keys/github-deploy.key \
  --path=kubernetes 
```

Create sops secret for flux:
```shell
kubectl create secret generic sops-age -n flux-system --from-file keys/age.agekey
```

Now flux will take over and deploy the rest of resources in this repository.
