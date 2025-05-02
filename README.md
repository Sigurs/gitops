# Sigurs' GitOps Repository
The main purpose is GitOps, but also contains Ansible IaaC for cluster setup.<br>
Combined for ease of use.

The long-term goal is to combine numerous independent docker-compose and kubernetes setups into 2-3 clusters for ease of management.


# Clusters

## Cluster: hcloud-cluster01
Cluster in Hetzner cloud. Publically accessible.<br>
Bit of a fast setup and not ideal.

## Cluster: home-k8s
Cluster located at home. No public access.<br>
HA is not the goal here due to different hardware.<br>
Utilizes Ansible.

Highlights:
- [UserNamespacesSupport](https://kubernetes.io/docs/tasks/configure-pod-container/user-namespaces/) is used for security due to some containers requiring root.

### Nodes
- home-k8s01
    - RaspberryPi 5 8GB
- home-k8s02 (upcoming)
    - VM running inside Proxmox
- xxx (upcoming)
    - Desktop with GPU resources.

### Services
#### Service:  AdGuard Home
Please not that AdGuard needs to be deployed with --server side flag.<br>
This is due to kubectl not being able to handle multiple protocols on the same port.<br>
Example: `kubectl apply --server-side --force-conflicts -n prod-adguard -k .`

Requires root on first start - or UserNamespaces.


#### Service:  Home Assistant
Configuration Migrated from a previous setup.<br>
Utilizes UserNamespaces to mitigate security concerns related to needing to be started as root.<br>
TODO: Bluetooth & other devices

#### Service: cert-manager
Used as a Certificate Authority for applications and for requesting Let's Encrypt certificates. <br>
Let's Encrypt ClusterIssuer yamls need to be manually filed and ran.

#### Service: system-upgrade
Utilize k3s' system-upgrade-controller to keep nodes up to date.

#### Service: Borgmatic
Used for backups.<br>
Normally I would consider using tools such as Velero to push backups to a s3 bucket, but this is more economical for me. <br>
Works great with k3s local-path provisioner.




# Usage
## Ansible

### Setup ansible
```bash
# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install ansible
python -m pip install ansible
ansible-galaxy collection install community.general kubernetes.core
```

### Run

```bash
# Activate virtual environment
source .venv/bin/activate

# Run ansible
ansible-playbook ansible/k3s-cluster/main.yaml -i home-k8s/ansible-inventory.yaml
```
