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
