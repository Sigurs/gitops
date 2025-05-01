# Sigurs' GitOps Repository
The main purpose is GitOps, but also contains ansible scripts for cluster setup.
Combined for ease of use.

## Clusters

### hcloud-cluster01
Cluster in Hetzner cloud. Publically accessible.

### home-k8s
Cluster located at home. No public access.

Highlights:
- [UserNamespacesSupport](https://kubernetes.io/docs/tasks/configure-pod-container/user-namespaces/) is used for security due to some containers requiring root.

#### AdGuard
Please not that AdGuard needs to be deployed with --server side flag.
This is due to kubectl not being able to handle multiple protocols on the same port.
Example: `kubectl apply --server-side --force-conflicts -n prod-adguard -k .`

## How to use
### Ansible

#### Setup ansible
```bash
python3 -m venv .venv
source .venv/bin/activate

python -m pip install ansible

```

#### Use

```bash
source .venv/bin/activate
ansible-playbook ansible/k3s-cluster/main.yaml -i home-k8s/ansible-inventory.yaml

```
