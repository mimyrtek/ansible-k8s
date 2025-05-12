# MongoDB Role for Proxmox Cloud

This Ansible role automates the deployment and configuration of MongoDB 8.0 with security features on Proxmox VMs.

## Role Structure

```
mongodb/
├── defaults/
│   └── main.yml               # Default variables
├── handlers/
│   └── main.yml               # Handlers for service restarts
├── meta/
│   └── main.yml               # Role metadata
├── tasks/
│   ├── create_admin_user.yml  # Create MongoDB admin user
│   ├── disable_auth.yml       # Disable MongoDB authentication
│   ├── enable_auth.yml        # Enable MongoDB authentication
│   ├── gen_tls.yml            # Generate TLS certificates
│   ├── install_mongodb.yml    # Install MongoDB
│   └── main.yml               # Main task file
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

The role executes tasks in the following order (defined in `tasks/main.yml`):

1. **Install MongoDB** (`install_mongodb.yml`):
   - Installs required packages (gnupg, curl, python3-pip)
   - Adds MongoDB 8.0 repository and GPG key
   - Installs MongoDB packages
   - Starts and enables the MongoDB service

2. **Generate TLS Certificates** (`gen_tls.yml`):
   - Creates TLS/SSL certificates for secure MongoDB connections
   - Configures MongoDB to use the certificates

3. **Disable Authentication** (`disable_auth.yml`):
   - Temporarily disables authentication to allow admin user creation
   - Restarts MongoDB service

4. **Create Admin User** (`create_admin_user.yml`):
   - Creates a MongoDB administrator user with appropriate privileges
   - Sets up initial database security

5. **Enable Authentication** (`enable_auth.yml`):
   - Re-enables authentication with the newly created admin user
   - Configures MongoDB for secure access
   - Restarts MongoDB service

## Usage

This role is designed to be used with the playbooks in the `playbook/` directory:

- `site.yaml`: Main playbook for deploying MongoDB with full security features
- `install-mongodb.yml`: Simplified playbook for basic MongoDB installation

### Example Playbook

```yaml
---
- name: Deploy MongoDB 8.0 with TLS and admin auth
  hosts: mongodb
  become: true
  roles:
    - mongodb
```

## Security Features

This role implements several security best practices:

1. **Authentication**: Creates and enables a dedicated admin user
2. **Encryption**: Sets up TLS/SSL for encrypted connections
3. **Repository Verification**: Uses GPG key verification for package installation

## License

MIT

## Author Information

Created for Proxmox Cloud environment.