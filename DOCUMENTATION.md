# Proxmox Cloud Ansible Documentation

This documentation provides a comprehensive overview of the Ansible automation setup for the Proxmox Cloud environment. The project includes roles and playbooks for setting up Kubernetes clusters and MongoDB instances.

## Directory Structure

```
/proxmox_cloud/ansible/
├── inventory/
│   └── hosts.ini                  # Inventory file with host definitions
├── k8srole/                       # Kubernetes setup role
│   ├── defaults/
│   │   └── main.yml               # Default variables
│   ├── files/                     # Static files
│   ├── handlers/
│   │   └── main.yml               # Handlers for service restarts
│   ├── meta/
│   │   └── main.yml               # Role metadata
│   ├── tasks/
│   │   ├── post_reboot.yml        # Tasks to run after system reboot
│   │   └── pre_reboot.yml         # Tasks to run before system reboot
│   ├── templates/                 # Jinja2 templates
│   ├── tests/                     # Role tests
│   ├── vars/
│   │   └── main.yml               # Role variables
│   └── README.md                  # Role documentation
├── mongodb/                       # MongoDB setup role
│   ├── defaults/
│   │   └── main.yml               # Default variables
│   ├── handlers/
│   │   └── main.yml               # Handlers for service restarts
│   ├── meta/
│   │   └── main.yml               # Role metadata
│   ├── tasks/
│   │   ├── create_admin_user.yml  # Create MongoDB admin user
│   │   ├── disable_auth.yml       # Disable MongoDB authentication
│   │   ├── enable_auth.yml        # Enable MongoDB authentication
│   │   ├── gen_tls.yml            # Generate TLS certificates
│   │   ├── install_mongodb.yml    # Install MongoDB
│   │   └── main.yml               # Main task file
│   └── README.md                  # Role documentation
└── playbook/                      # Playbooks directory
    ├── install-mongodb.yml        # MongoDB installation playbook
    ├── k8s-join.yaml              # Kubernetes node join playbook
    ├── k8s.yaml                   # Kubernetes cluster setup playbook
    └── site.yaml                  # Main site playbook
```

## Inventory Configuration

The inventory file (`hosts.ini`) defines the following host groups:

- **k8sctrl**: Kubernetes control plane node (10.10.50.150)
- **k8snodes**: Kubernetes worker nodes (10.10.50.151-153)
- **mongodb**: MongoDB server (10.10.50.234)

All hosts use the `ubuntu` user with SSH key authentication.

## Roles

### 1. k8srole - Kubernetes Cluster Setup

This role automates the setup of a Kubernetes cluster with one control plane node and multiple worker nodes.

#### Tasks

**Pre-reboot Tasks (`pre_reboot.yml`):**
- Install required packages (apt-transport-https, ca-certificates, curl, gpg, nfs-common)
- Configure Kubernetes apt repository and key
- Install and configure containerd as the container runtime
- Disable swap (required for Kubernetes)
- Configure system settings for Kubernetes (IP forwarding, br_netfilter module)
- Reboot the system to apply changes

**Post-reboot Tasks (`post_reboot.yml`):**
- Install Kubernetes packages (kubeadm, kubectl, kubelet)
- Initialize Kubernetes cluster on the control plane node
- Set up kubeconfig for the root user
- Install Flannel CNI for pod networking
- Install k9s (Kubernetes CLI management tool)
- Disable unattended upgrades

### 2. mongodb - MongoDB Setup

This role automates the installation and configuration of MongoDB with security features.

#### Tasks

**Main Tasks Flow:**
1. Install MongoDB (`install_mongodb.yml`)
2. Generate TLS certificates (`gen_tls.yml`)
3. Temporarily disable authentication (`disable_auth.yml`)
4. Create admin user (`create_admin_user.yml`)
5. Enable authentication (`enable_auth.yml`)

## Playbooks

### 1. k8s.yaml

This playbook sets up a Kubernetes cluster by:
1. Running pre-reboot tasks on all Kubernetes nodes
2. Running post-reboot tasks to complete the Kubernetes setup

### 2. k8s-join.yaml

This playbook handles joining worker nodes to the Kubernetes cluster:
1. Retrieves the join command from the control plane node
2. Waits for the Kubernetes API to become available
3. Joins worker nodes to the cluster using the retrieved command

### 3. install-mongodb.yml

This playbook provides a standalone MongoDB installation:
- Installs dependencies
- Adds MongoDB 8.0 repository and GPG key
- Installs MongoDB packages
- Enables and starts the MongoDB service

### 4. site.yaml

The main site playbook that deploys MongoDB 8.0 with TLS and admin authentication by applying the mongodb role.

## Kubernetes Cluster Configuration

The Kubernetes cluster is configured with:
- Control plane endpoint: 10.10.50.150
- Pod network CIDR: 10.200.0.0/16
- Network plugin: Flannel

## MongoDB Configuration

MongoDB is installed with:
- Version: 8.0 Community Edition
- Authentication: Enabled
- TLS/SSL: Enabled with generated certificates
- Admin user: Created during deployment

## Usage Instructions

### Setting up a Kubernetes Cluster

```bash
# Deploy the full Kubernetes cluster
ansible-playbook -i inventory/hosts.ini playbook/k8s.yaml

# Join worker nodes to an existing cluster
ansible-playbook -i inventory/hosts.ini playbook/k8s-join.yaml
```

### Setting up MongoDB

```bash
# Deploy MongoDB with full security features
ansible-playbook -i inventory/hosts.ini playbook/site.yaml

# Deploy MongoDB standalone installation
ansible-playbook -i inventory/hosts.ini playbook/install-mongodb.yml
```

## Security Considerations

1. **Kubernetes**:
   - Uses secure apt repositories with GPG verification
   - Configures proper network security with Flannel
   - Sets up proper authentication for the cluster

2. **MongoDB**:
   - Implements TLS/SSL encryption
   - Creates dedicated admin user
   - Enables authentication

## Maintenance and Troubleshooting

### Common Issues

1. **Kubernetes Pod Network Issues**:
   - Check Flannel configuration
   - Verify pod CIDR configuration (10.200.0.0/16)

2. **MongoDB Connection Issues**:
   - Verify TLS certificate validity
   - Check authentication credentials
   - Ensure MongoDB service is running

### Updating Components

1. **Kubernetes Updates**:
   - Update apt repositories
   - Use kubeadm for controlled upgrades

2. **MongoDB Updates**:
   - Follow standard MongoDB upgrade procedures
   - Back up data before upgrades

## Future Improvements

1. Add monitoring and logging solutions
2. Implement backup and restore automation
3. Add high availability configurations for both Kubernetes and MongoDB
4. Implement infrastructure as code for the underlying Proxmox environment