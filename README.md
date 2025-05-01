# Sigurs' GitOps Repository
The main purpose is GitOps, but also contains Ansible IaaC for cluster setup.<br>
Combined for ease of use.

The long-term goal is to combine numerous independent docker-compose and kubernetes setups into 2-3 clusters for ease of management.


## Clusters

### hcloud-cluster01
Cluster in Hetzner cloud. Publically accessible.

### home-k8s
Cluster located at home. No public access.

Highlights:
- [UserNamespacesSupport](https://kubernetes.io/docs/tasks/configure-pod-container/user-namespaces/) is used for security due to some containers requiring root.

#### Service:  AdGuard Home
Please not that AdGuard needs to be deployed with --server side flag.<br>
This is due to kubectl not being able to handle multiple protocols on the same port.<br>
Example: `kubectl apply --server-side --force-conflicts -n prod-adguard -k .`

Requires root on first start - or UserNamespaces.


#### Service:  Home Assistant
Configuration Migrated from a previous setup.<br>
Utilizes UserNamespaces to mitigate security concerns related to needing to be started as root.<br>
TODO: Bluetooth & other devices

## How to use
### Ansible

#### Setup ansible
```bash
# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install ansible
python -m pip install ansible
```

#### Use

```bash
# Activate virtual environment
source .venv/bin/activate

# Run ansible
ansible-playbook ansible/k3s-cluster/main.yaml -i home-k8s/ansible-inventory.yaml
```
