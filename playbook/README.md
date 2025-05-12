# Ansible Playbooks for Proxmox Cloud

This directory contains Ansible playbooks for deploying and configuring services in the Proxmox Cloud environment.

## Available Playbooks

### 1. k8s.yaml - Kubernetes Cluster Setup

This playbook sets up a complete Kubernetes cluster with one control plane node and multiple worker nodes.

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

**Usage:**
```bash
ansible-playbook -i ../inventory/hosts.ini k8s.yaml
```

### 2. k8s-join.yaml - Join Nodes to Kubernetes Cluster

This playbook automates the process of joining worker nodes to an existing Kubernetes cluster.

```yaml
---
# First play: Get join command from control plane
- name: Retrieve kubeadm join command from master (k8sctrl)
  hosts: k8sctrl
  become: yes
  gather_facts: false
  tasks:
    - name: Generate kubeadm join command on master node
      command: kubeadm token create --print-join-command
      register: join_command_output
      changed_when: false
      run_once: true

    - name: Save join command as fact on k8sctrl
      set_fact:
        kubeadm_join_command: "{{ join_command_output.stdout }}"
      run_once: true

# Second play: Join worker nodes to the cluster
- name: Join worker nodes to the Kubernetes cluster
  hosts: k8snodes
  become: yes
  gather_facts: false
  tasks:
    - name: Wait for the Kubernetes API to become available
      wait_for:
        host: "10.10.50.150"   # control plane endpoint IP
        port: 6443
        delay: 10
        timeout: 300

    - name: Join worker node to the cluster
      shell: "{{ hostvars['k8sctrl']['kubeadm_join_command'] }}"
```

**Usage:**
```bash
ansible-playbook -i ../inventory/hosts.ini k8s-join.yaml
```

### 3. install-mongodb.yml - MongoDB Installation

This playbook provides a standalone installation of MongoDB 8.0 Community Edition.

```yaml
---
- name: Install MongoDB 8.0 Community Edition on Ubuntu 22.04
  hosts: all
  become: true

  tasks:
    - name: Ensure dependencies are installed
      apt:
        name:
          - gnupg
          - curl
          - ca-certificates
        state: present
        update_cache: yes

    # Additional tasks for MongoDB installation...
```

**Usage:**
```bash
ansible-playbook -i ../inventory/hosts.ini install-mongodb.yml -l mongodb
```

### 4. site.yaml - Full MongoDB Deployment

This playbook deploys MongoDB 8.0 with TLS and admin authentication by applying the mongodb role.

```yaml
---
- name: Deploy MongoDB 8.0 with TLS and admin auth
  hosts: all
  become: true
  roles:
    - mongodb
```

**Usage:**
```bash
ansible-playbook -i ../inventory/hosts.ini site.yaml -l mongodb
```

## Common Options

- `-i ../inventory/hosts.ini`: Specify the inventory file
- `-l [group]`: Limit execution to specific host group
- `-v`: Increase verbosity for debugging
- `--check`: Run in check mode (dry run)
- `--diff`: Show differences in changed files

## Prerequisites

- Ansible 2.9+
- SSH access to target hosts
- Proper inventory configuration in `../inventory/hosts.ini`

## Notes

- The Kubernetes playbooks assume Ubuntu 22.04 LTS hosts
- MongoDB playbooks are configured for MongoDB 8.0 Community Edition
- All playbooks require root privileges (using `become: true`)

## License

MIT

## Author Information

Created for Proxmox Cloud environment.