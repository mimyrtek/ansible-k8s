# Kubernetes Role for Proxmox Cloud

This Ansible role automates the deployment and configuration of a Kubernetes cluster on Proxmox VMs.

## Role Structure

```
k8srole/
├── defaults/
│   └── main.yml               # Default variables
├── files/                     # Static files
├── handlers/
│   └── main.yml               # Handlers for service restarts
├── meta/
│   └── main.yml               # Role metadata
├── tasks/
│   ├── post_reboot.yml        # Tasks to run after system reboot
│   └── pre_reboot.yml         # Tasks to run before system reboot
├── templates/                 # Jinja2 templates
├── tests/                     # Role tests
└── vars/
    └── main.yml               # Role variables
```

## Requirements

- Ubuntu 22.04 LTS target hosts
- SSH access to target hosts
- Ansible 2.9+

## Role Variables

Variables are defined in `defaults/main.yml` and can be overridden as needed.

## Dependencies

None.

## Task Workflows

### Pre-reboot Tasks

The `pre_reboot.yml` tasks handle:

1. Installing required system packages:
   - apt-transport-https
   - ca-certificates
   - curl
   - gpg
   - nfs-common

2. Setting up Kubernetes repositories:
   - Adding Kubernetes apt key
   - Configuring apt repository for Kubernetes 1.30

3. Configuring containerd:
   - Installing containerd package
   - Creating default configuration
   - Enabling SystemdCgroup

4. Preparing the system for Kubernetes:
   - Disabling swap (required for Kubernetes)
   - Configuring IP forwarding
   - Loading br_netfilter module
   - Rebooting the system to apply changes

### Post-reboot Tasks

The `post_reboot.yml` tasks handle:

1. Installing Kubernetes components:
   - kubeadm
   - kubectl
   - kubelet

2. Initializing the Kubernetes cluster:
   - Setting control plane endpoint to 10.10.50.150
   - Configuring pod network CIDR as 10.200.0.0/16
   - Setting up kubeconfig for the root user

3. Configuring networking:
   - Installing Flannel CNI plugin
   - Configuring pod network CIDR

4. Installing additional tools:
   - k9s for cluster management

5. System configuration:
   - Disabling unattended upgrades

## Usage

This role is designed to be used with the playbooks in the `playbook/` directory:

- `k8s.yaml`: Main playbook for setting up the Kubernetes cluster
- `k8s-join.yaml`: Playbook for joining worker nodes to an existing cluster

### Example Playbook

```yaml
---
- hosts: k8sctrl,k8snodes
  become: true
  tasks:
    - import_role:
        name: k8srole
        tasks_from: pre_reboot.yml

- hosts: k8sctrl,k8snodes
  become: true
  tasks:
    - import_role:
        name: k8srole
        tasks_from: post_reboot.yml
```

## License

MIT

## Author Information

Created for Proxmox Cloud environment.