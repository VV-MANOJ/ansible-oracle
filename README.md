# VMware Cloud Director VM Creation Ansible Role

This Ansible role creates Virtual Machines in VMware Cloud Director (VCD) 10.4 using REST APIs version 39.0. The role follows Ansible best practices with comprehensive error handling, variable validation, and modular task organization.


## Features

- ✅ VMware Cloud Director 10.4 support with REST API v39.0
- ✅ Complete VM lifecycle management (create, configure, power on)
- ✅ Advanced network configuration support
- ✅ Comprehensive error handling and cleanup
- ✅ Variable validation and input checking
- ✅ Modular task organization
- ✅ Support for custom storage profiles
- ✅ Asynchronous task monitoring
- ✅ Idempotent operations
- ✅ Secure credential handling

## Requirements

### System Requirements
- Ansible >= 2.9
- Python >= 3.6
- Access to VMware Cloud Director 10.4
- Network connectivity to VCD API endpoints

### Ansible Collections
```bash
# Install required collections
ansible-galaxy collection install community.general
ansible-galaxy collection install ansible.posix
```

### Python Dependencies
```bash
# Install required Python packages
pip install requests
pip install lxml
```

## Installation

### Step 1: Clone Repository
```bash
# Clone the repository
git clone https://github.com/your-username/ansible-vcd-vm-role.git
cd ansible-vcd-vm-role

# Verify structure
tree roles/vcd_vm_create/
```

### Step 2: Install Dependencies
```bash
# Install required collections
ansible-galaxy collection install community.general
ansible-galaxy collection install ansible.posix

# Install Python dependencies
pip install requests lxml
```

## Directory Structure

```
ansible-vcd-vm-role/
├── README.md
├── ansible.cfg
├── inventory
├── group_vars/
│   └── vcd_environment.yml
├── host_vars/
│   └── vm1.yml
├── playbooks/
│   └── create_vcd_vm.yml
├── roles/
│   └── vcd_vm_create/
│       ├── defaults/
│       │   └── main.yml
│       ├── files/
│       ├── handlers/
│       ├── meta/
│       │   └── main.yml
│       ├── tasks/
│       │   ├── main.yml
│       │   ├── validate_vars.yml
│       │   ├── authenticate.yml
│       │   ├── get_org_details.yml
│       │   ├── get_vdc_details.yml
│       │   ├── get_catalog_template.yml
│       │   ├── create_vm.yml
│       │   ├── configure_network.yml
│       │   ├── power_on_vm.yml
│       │   ├── wait_task.yml
│       │   ├── wait_vm_ready.yml
│       │   ├── error_handler.yml
│       │   └── cleanup.yml
│       ├── templates/
│       └── vars/
│           └── main.yml
└── vault/
    └── credentials.yml
```

## Configuration

### Step 1: Configure Ansible Settings
The repository includes a pre-configured `ansible.cfg` file with optimal settings.

### Step 2: Set Up Inventory
The repository includes a basic inventory file. Modify if needed:
```bash
# Edit inventory file
vi inventory
```

### Step 3: Create Vault for Credentials
```bash
# Create encrypted vault file
ansible-vault create vault/credentials.yml
```

Add the following content to the vault:
```yaml
vault_vcd_password: "your_vcd_password_here"
vault_vcd_username: "your_vcd_username_here"
```

## Usage

### Basic VM Creation

#### Step 1: Configure Variables
Edit `group_vars/vcd_environment.yml`:
```yaml
---
# VCD Connection Settings
vcd_host: "your-vcd-hostname.com"
vcd_org: "YourOrganization"
vcd_vdc: "YourVDC"
vcd_validate_certs: true

# Default VM Settings
vcd_vm_catalog: "YourCatalog"
vcd_vm_template: "YourTemplate"
vcd_vm_cpu_count: 2
vcd_vm_memory_mb: 4096
```

#### Step 2: Create Playbook
```bash
cat > playbooks/create_vm.yml << 'EOF'
---
- name: "Create VM in VMware Cloud Director"
  hosts: localhost
  connection: local
  gather_facts: false
  vars_files:
    - ../vault/credentials.yml
  vars:
    vcd_username: "{{ vault_vcd_username }}"
    vcd_password: "{{ vault_vcd_password }}"
    vcd_vm_name: "test-vm-001"
    vcd_vm_description: "Test VM created by Ansible"
  
  roles:
    - role: vcd_vm_create
EOF
```

#### Step 3: Execute Playbook
```bash
# Run the playbook
ansible-playbook playbooks/create_vm.yml --ask-vault-pass

# Run with verbose output for debugging
ansible-playbook playbooks/create_vm.yml --ask-vault-pass -vvv

# Run with specific inventory
ansible-playbook -i inventory playbooks/create_vm.yml --ask-vault-pass
```

### Advanced VM Creation with Network Configuration

```bash
cat > playbooks/create_vm_advanced.yml << 'EOF'
---
- name: "Create Advanced VM with Network Configuration"
  hosts: localhost
  connection: local
  gather_facts: false
  vars_files:
    - ../vault/credentials.yml
  vars:
    vcd_username: "{{ vault_vcd_username }}"
    vcd_password: "{{ vault_vcd_password }}"
    vcd_vm_name: "web-server-prod"
    vcd_vm_description: "Production web server"
    vcd_vm_cpu_count: 4
    vcd_vm_memory_mb: 8192
    
    # Network Configuration
    vcd_vm_network_config:
      - network_name: "production-network"
        is_inherited: false
        ip_allocation_mode: "STATIC"
        ip_address: "192.168.100.10"
        gateway: "192.168.100.1"
        netmask: "255.255.255.0"
        dns1: "8.8.8.8"
        dns2: "8.8.4.4"
        fence_mode: "bridged"
    
    # Control Settings
    vcd_vm_power_on: true
    vcd_vm_wait_ready: true
    vcd_cleanup_on_error: true
  
  roles:
    - role: vcd_vm_create
EOF
```

### Bulk VM Creation

```bash
cat > playbooks/create_multiple_vms.yml << 'EOF'
---
- name: "Create Multiple VMs"
  hosts: localhost
  connection: local
  gather_facts: false
  vars_files:
    - ../vault/credentials.yml
  vars:
    vcd_username: "{{ vault_vcd_username }}"
    vcd_password: "{{ vault_vcd_password }}"
    
    vm_list:
      - name: "web-01"
        cpu: 4
        memory: 8192
        ip: "192.168.1.10"
      - name: "web-02"
        cpu: 4
        memory: 8192
        ip: "192.168.1.11"
      - name: "db-01"
        cpu: 8
        memory: 16384
        ip: "192.168.1.20"

  tasks:
    - name: "Create VMs from list"
      include_role:
        name: vcd_vm_create
      vars:
        vcd_vm_name: "{{ item.name }}"
        vcd_vm_cpu_count: "{{ item.cpu }}"
        vcd_vm_memory_mb: "{{ item.memory }}"
        vcd_vm_network_config:
          - network_name: "production-network"
            is_inherited: false
            ip_allocation_mode: "STATIC"
            ip_address: "{{ item.ip }}"
            gateway: "192.168.1.1"
            netmask: "255.255.255.0"
      loop: "{{ vm_list }}"
EOF
```

## Variables

### Required Variables
| Variable | Description | Example |
|----------|-------------|---------|
| `vcd_host` | VCD hostname/IP | `vcd.example.com` |
| `vcd_username` | VCD username | `admin` |
| `vcd_password` | VCD password | `password123` |
| `vcd_org` | VCD organization | `MyOrg` |
| `vcd_vdc` | VCD virtual datacenter | `MyVDC` |
| `vcd_vm_name` | VM name | `my-test-vm` |
| `vcd_vm_catalog` | Catalog name | `MyCatalog` |
| `vcd_vm_template` | Template name | `Ubuntu-20.04` |
| `vcd_vm_cpu_count` | Number of CPUs | `4` |
| `vcd_vm_memory_mb` | Memory in MB | `8192` |

### Optional Variables
| Variable | Description | Default |
|----------|-------------|---------|
| `vcd_validate_certs` | Validate SSL certificates | `true` |
| `vcd_api_timeout` | API timeout in seconds | `30` |
| `vcd_vm_description` | VM description | `VM created by Ansible` |
| `vcd_vm_power_on` | Power on after creation | `true` |
| `vcd_vm_wait_ready` | Wait for VM to be ready | `true` |
| `vcd_cleanup_on_error` | Cleanup on error | `false` |

### Network Configuration Variables
```yaml
vcd_vm_network_config:
  - network_name: "network-name"          # Required
    is_inherited: true                    # Optional, default: true
    ip_allocation_mode: "DHCP"           # Optional: DHCP, STATIC, POOL
    ip_address: "192.168.1.100"         # Optional, for STATIC mode
    gateway: "192.168.1.1"              # Optional
    netmask: "255.255.255.0"            # Optional
    dns1: "8.8.8.8"                     # Optional
    dns2: "8.8.4.4"                     # Optional
    fence_mode: "bridged"               # Optional: bridged, isolated, natRouted
```

## Examples

### Example 1: Simple VM Creation
```bash
ansible-playbook playbooks/create_vm.yml \
  --ask-vault-pass \
  -e "vcd_vm_name=simple-test-vm" \
  -e "vcd_vm_cpu_count=2" \
  -e "vcd_vm_memory_mb=4096"
```

### Example 2: VM with Static IP
```bash
ansible-playbook playbooks/create_vm.yml \
  --ask-vault-pass \
  -e "vcd_vm_name=static-ip-vm" \
  -e '{
    "vcd_vm_network_config": [
      {
        "network_name": "production-network",
        "is_inherited": false,
        "ip_allocation_mode": "STATIC",
        "ip_address": "192.168.1.50"
      }
    ]
  }'
```

### Example 3: High-Resource VM
```bash
ansible-playbook playbooks/create_vm.yml \
  --ask-vault-pass \
  -e "vcd_vm_name=high-resource-vm" \
  -e "vcd_vm_cpu_count=8" \
  -e "vcd_vm_memory_mb=32768" \
  -e "vcd_vm_template=CentOS-8-Template"
```

## Execution Commands Summary

### Pre-requisites
```bash
# 1. Install Ansible (if not already installed)
pip install ansible

# 2. Clone the repository
git clone https://github.com/your-username/ansible-vcd-vm-role.git
cd ansible-vcd-vm-role

# 3. Install required collections
ansible-galaxy collection install community.general

# 4. Create vault for credentials
ansible-vault create vault/credentials.yml
```

### Basic Execution
```bash
# Simple VM creation
ansible-playbook playbooks/create_vm.yml --ask-vault-pass

# With custom variables
ansible-playbook playbooks/create_vm