# Proxmox Cloud Ansible Usage Guide

This guide provides step-by-step instructions for using the Ansible automation to deploy and manage Kubernetes clusters and MongoDB instances in the Proxmox Cloud environment.

## Prerequisites

Before using these Ansible playbooks, ensure you have:

1. **Ansible Installation**:
   ```bash
   sudo apt update
   sudo apt install ansible
   ```

2. **SSH Access**:
   - SSH key pair generated and configured
   - SSH key added to all target hosts
   - Ability to connect to all hosts defined in the inventory

3. **Target Hosts**:
   - Ubuntu 22.04 LTS installed on all target VMs
   - Network connectivity between all hosts
   - Sufficient resources (CPU, RAM, disk) for Kubernetes and/or MongoDB

## Getting Started

### 1. Clone the Repository

```bash
git clone <repository-url>
cd proxmox_cloud/ansible
```

### 2. Review and Update Inventory

The inventory file is located at `inventory/hosts.ini`. Review and update it to match your environment:

```ini
[k8sctrl]
k8sctrl ansible_host=10.10.50.150 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_ed25519

[k8snodes]
k8snode1 ansible_host=10.10.50.151 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_ed25519
k8snode2 ansible_host=10.10.50.152 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_ed25519
k8snode3 ansible_host=10.10.50.153 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_ed25519

[mongodb]
mongodb ansible_host=10.10.50.234 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

Modify:
- IP addresses to match your VM IPs
- `ansible_user` if using a different username
- `ansible_ssh_private_key_file` to point to your SSH key

### 3. Verify Connectivity

Test SSH connectivity to all hosts:

```bash
ansible all -i inventory/hosts.ini -m ping
```

You should see a successful response from all hosts.

## Deploying Kubernetes Cluster

### 1. Deploy the Full Kubernetes Cluster

```bash
cd playbook
ansible-playbook -i ../inventory/hosts.ini k8s.yaml
```

This playbook:
1. Prepares all nodes with necessary packages and configurations
2. Reboots all nodes to apply system changes
3. Installs Kubernetes components
4. Initializes the cluster on the control plane node
5. Sets up Flannel networking

The process takes approximately 10-15 minutes to complete.

### 2. Verify Kubernetes Cluster

SSH into the control plane node:

```bash
ssh ubuntu@10.10.50.150
```

Check cluster status:

```bash
sudo kubectl get nodes
```

You should see the control plane node in "Ready" status.

### 3. Join Worker Nodes

If worker nodes didn't join automatically, run:

```bash
cd playbook
ansible-playbook -i ../inventory/hosts.ini k8s-join.yaml
```

This playbook:
1. Retrieves the join command from the control plane
2. Executes the join command on all worker nodes

### 4. Verify All Nodes

SSH into the control plane node and check:

```bash
sudo kubectl get nodes
```

All nodes should now show "Ready" status.

## Deploying MongoDB

### 1. Deploy MongoDB with Full Security

```bash
cd playbook
ansible-playbook -i ../inventory/hosts.ini site.yaml -l mongodb
```

This playbook:
1. Installs MongoDB 8.0
2. Generates TLS certificates
3. Creates an admin user
4. Enables authentication and TLS

### 2. Alternative: Basic MongoDB Installation

For a simpler MongoDB installation without TLS and authentication:

```bash
cd playbook
ansible-playbook -i ../inventory/hosts.ini install-mongodb.yml -l mongodb
```

### 3. Verify MongoDB Installation

SSH into the MongoDB server:

```bash
ssh ubuntu@10.10.50.234
```

Check MongoDB status:

```bash
sudo systemctl status mongod
```

Connect to MongoDB:

```bash
mongosh
```

## Common Operations

### Checking Playbook Syntax

```bash
ansible-playbook -i inventory/hosts.ini playbook/k8s.yaml --syntax-check
```

### Dry Run (Check Mode)

```bash
ansible-playbook -i inventory/hosts.ini playbook/k8s.yaml --check
```

### Running with Increased Verbosity

```bash
ansible-playbook -i inventory/hosts.ini playbook/k8s.yaml -v
```

### Targeting Specific Hosts

```bash
ansible-playbook -i inventory/hosts.ini playbook/k8s.yaml -l k8sctrl
```

## Troubleshooting

### 1. SSH Connection Issues

If you encounter SSH connection problems:

```bash
# Check SSH manually
ssh -i ~/.ssh/id_ed25519 ubuntu@10.10.50.150

# Verify inventory file paths and permissions
chmod 600 ~/.ssh/id_ed25519
```

### 2. Kubernetes Initialization Failures

If `kubeadm init` fails:

```bash
# Check logs on control plane node
ssh ubuntu@10.10.50.150
sudo kubeadm reset
sudo rm -rf /etc/kubernetes/
sudo rm -rf ~/.kube/
```

Then run the playbook again.

### 3. Pod Network Issues

If pods can't communicate:

```bash
# Check Flannel pods
kubectl get pods -n kube-system | grep flannel

# Check Flannel logs
kubectl logs -n kube-system <flannel-pod-name>
```

### 4. MongoDB Connection Issues

If you can't connect to MongoDB:

```bash
# Check MongoDB service
sudo systemctl status mongod

# Check MongoDB logs
sudo cat /var/log/mongodb/mongod.log

# Verify firewall settings
sudo ufw status
```

## Maintenance

### Updating Kubernetes

```bash
# On control plane node
sudo apt update
sudo apt-cache madison kubeadm
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm=<version> && sudo apt-mark hold kubeadm
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v<version>
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get install -y kubelet=<version> kubectl=<version> && sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

### Updating MongoDB

```bash
sudo apt update
sudo apt install mongodb-org
sudo systemctl restart mongod
```

## Conclusion

This guide covers the basic usage of the Ansible playbooks for deploying and managing Kubernetes and MongoDB in the Proxmox Cloud environment. For more detailed information, refer to the documentation for each role and playbook.