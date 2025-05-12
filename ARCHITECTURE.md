# Proxmox Cloud Ansible Architecture

This document provides a detailed architectural overview of the Ansible automation setup for the Proxmox Cloud environment.

## System Architecture

The Ansible automation is designed to configure and manage two primary systems:

1. **Kubernetes Cluster**: A multi-node Kubernetes cluster with one control plane and multiple worker nodes
2. **MongoDB Database**: A secure MongoDB instance with authentication and TLS encryption

### Network Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Proxmox Cloud                           │
│                                                             │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐    │
│  │  k8sctrl    │     │  k8snode1   │     │  k8snode2   │    │
│  │10.10.50.150 │────▶│10.10.50.151 │────▶│10.10.50.152 │    │
│  └─────────────┘     └─────────────┘     └─────────────┘    │
│        │                                       │            │
│        │                                       │            │
│        ▼                                       ▼            │
│  ┌─────────────┐                        ┌─────────────┐     │
│  │  k8snode3   │                        │  MongoDB    │     │
│  │10.10.50.153 │                        │10.10.50.234 │     │
│  └─────────────┘                        └─────────────┘     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Component Architecture

### 1. Kubernetes Cluster

The Kubernetes cluster consists of:

- **Control Plane Node (k8sctrl)**:
  - Hosts the Kubernetes API server, scheduler, and controller manager
  - Acts as the central management point for the cluster
  - IP: 10.10.50.150

- **Worker Nodes (k8snode1, k8snode2, k8snode3)**:
  - Run containerized applications
  - IPs: 10.10.50.151, 10.10.50.152, 10.10.50.153

- **Network Configuration**:
  - Control plane endpoint: 10.10.50.150
  - Pod network CIDR: 10.200.0.0/16
  - Network plugin: Flannel

### 2. MongoDB Database

- **MongoDB Server**:
  - Runs MongoDB 8.0 Community Edition
  - IP: 10.10.50.234
  - Security features:
    - TLS/SSL encryption
    - Authentication enabled
    - Admin user with restricted privileges

## Automation Architecture

The Ansible automation is structured using roles and playbooks:

### Roles

1. **k8srole**: Handles Kubernetes cluster setup
   - Pre-reboot tasks: System preparation, containerd setup
   - Post-reboot tasks: Kubernetes installation, cluster initialization

2. **mongodb**: Handles MongoDB installation and configuration
   - Installation
   - TLS certificate generation
   - User management
   - Security configuration

### Playbooks

1. **k8s.yaml**: Orchestrates the Kubernetes cluster setup
   - Executes pre-reboot tasks
   - Executes post-reboot tasks

2. **k8s-join.yaml**: Manages worker node joining
   - Retrieves join command from control plane
   - Executes join on worker nodes

3. **install-mongodb.yml**: Basic MongoDB installation
   - Package installation
   - Service configuration

4. **site.yaml**: Full MongoDB deployment
   - Applies the mongodb role for complete setup

## Workflow Diagrams

### Kubernetes Cluster Setup Workflow

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Install Deps   │     │ Configure       │     │ Disable Swap    │
│  & Repositories │────▶│ Containerd      │────▶│ & System Config │
└─────────────────┘     └─────────────────┘     └────────┬────────┘
                                                         │
                                                         ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Install K9s    │     │ Install         │     │ System Reboot   │
│  & Flannel      │◀────│ Kubernetes      │◀────│                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │
        │
        ▼
┌─────────────────┐     ┌─────────────────┐
│  Initialize     │     │ Join Worker     │
│  Control Plane  │────▶│ Nodes           │
└─────────────────┘     └─────────────────┘
```

### MongoDB Setup Workflow

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Install        │     │ Generate        │     │ Disable         │
│  MongoDB        │────▶│ TLS Certs       │────▶│ Authentication  │
└─────────────────┘     └─────────────────┘     └────────┬────────┘
                                                         │
                                                         ▼
                                                ┌─────────────────┐
                                                │ Create Admin    │
                                                │ User            │
                                                └────────┬────────┘
                                                         │
                                                         ▼
                                                ┌─────────────────┐
                                                │ Enable          │
                                                │ Authentication  │
                                                └─────────────────┘
```

## Security Architecture

### Kubernetes Security

1. **Network Security**:
   - Pod network isolation with Flannel
   - Control plane endpoint security

2. **Authentication & Authorization**:
   - Kubeconfig-based authentication
   - Role-based access control (RBAC)

3. **System Security**:
   - Secure apt repositories with GPG verification
   - Containerd with SystemdCgroup for container isolation

### MongoDB Security

1. **Transport Security**:
   - TLS/SSL encryption for all connections

2. **Authentication**:
   - Admin user with password authentication
   - Role-based access control

3. **System Security**:
   - Repository verification with GPG keys
   - Principle of least privilege for service accounts

## Conclusion

The Proxmox Cloud Ansible architecture provides a comprehensive automation solution for deploying and managing Kubernetes clusters and MongoDB databases in a secure and repeatable manner. The modular design with roles and playbooks allows for flexibility and maintainability of the infrastructure automation.