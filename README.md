# Ansible Role: Proxmox Node Discovery

An Ansible role that discovers Proxmox VE nodes, containers (LXC), and virtual machines (QEMU) via the Proxmox API. 
This role automatically populates host variables with Proxmox-related information, including node assignment, VM/container type, status, MAC address, QEMU agent status, and resource allocation.

## Features

- Discover Proxmox nodes, containers, and virtual machines via the Proxmox API
- Populate host variables with Proxmox-related information
- Flexible configuration options to enable/disable specific features
- The current implementation only support one Proxmox cluster

## Requirements

- Ansible 2.9 or higher
- Proxmox VE 6.x or higher
- Proxmox API access (token-based authentication recommended)

## Role Variables

### Required Variables

```yaml
# Proxmox API connection details
proxmox_host: "your-proxmox-host.example.com"  # Proxmox API host
proxmox_user: "user@pve!token-name"            # API token user (format: user@realm!token-name)
proxmox_pass: "your-token-secret"              # API token secret
```

### Optional Variables

```yaml
# Feature toggles (all disabled by default)
discovery_proxmox_macaddress: false    # Enable MAC address discovery
discovery_proxmox_qemu: false          # Enable QEMU agent status discovery
discovery_proxmox_resources: false     # Enable resource (cores/memory) discovery

# Other options
discovery_timeout: 30                  # API request timeout in seconds
proxmox_verify_ssl: false              # Whether to verify SSL certificates
```

## Host Variables

Each host in your inventory should have a `proxmox_id` variable defined to identify the corresponding VM or container in Proxmox:

```ini
[proxmox_vms]
vm1 ansible_host=192.168.1.101 proxmox_id=101
vm2 ansible_host=192.168.1.102 proxmox_id=102

[proxmox_lxcs]
lxc1 ansible_host=192.168.1.201 proxmox_id=201
lxc2 ansible_host=192.168.1.202 proxmox_id=202
```

## Variables Set by the Role

This role sets the following host variables for each host that has a matching VM or container in Proxmox:

- `proxmox_node`: The Proxmox node where the VM/container is located
- `proxmox_type`: The type of the VM/container (`vm` or `lxc`)
- `proxmox_status`: The current status of the VM/container (e.g., `running`, `stopped`)
- `proxmox_cores`: Number of CPU cores (when `discovery_proxmox_resources` is enabled)
- `proxmox_memory`: Memory allocation in MB (when `discovery_proxmox_resources` is enabled)
- `proxmox_qemu`: QEMU agent status (when `discovery_proxmox_qemu` is enabled)
- `macaddress`: MAC address (when `discovery_proxmox_macaddress` is enabled)

## Example Playbook

```yaml
---
- name: Discover Proxmox VMs and containers
  hosts: all
  gather_facts: true
  vars:
    proxmox_host: "proxmox.example.com"
    proxmox_user: "ansible@pve!ansible-token"
    proxmox_pass: "{{ vault_proxmox_token }}"
  
  tasks:
    - name: Include Proxmox node discovery role
      include_role:
        name: ansible-role-proxmox-node-discovery
      vars:
        discovery_proxmox_macaddress: true
        discovery_proxmox_qemu: true
        discovery_proxmox_resources: true
      when: 
        - proxmox_host is defined
        - proxmox_user is defined
        - proxmox_pass is defined
```

## Inventory Example

### INI Format

```ini
[proxmox_hosts]
proxmox.example.com

[vms]
vm1 ansible_host=192.168.1.101 proxmox_id=101
vm2 ansible_host=192.168.1.102 proxmox_id=102

[containers]
lxc1 ansible_host=192.168.1.201 proxmox_id=201
lxc2 ansible_host=192.168.1.202 proxmox_id=202
```

### YAML Format

```yaml
all:
  children:
    proxmox_hosts:
      hosts:
        proxmox.example.com:
    vms:
      hosts:
        vm1:
          ansible_host: 192.168.1.101
          proxmox_id: 101
        vm2:
          ansible_host: 192.168.1.102
          proxmox_id: 102
    containers:
      hosts:
        lxc1:
          ansible_host: 192.168.1.201
          proxmox_id: 201
        lxc2:
          ansible_host: 192.168.1.202
          proxmox_id: 202
```

## Testing

Due to the requirement of a Proxmox server, automated testing is challenging. Manual testing can be performed using:

```bash
ansible-playbook -i inventory.ini test-playbook.yml -v
```

## License

MIT

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin feature/my-new-feature`)
5. Create a new Pull Request

## Author Information

Lucas Janin
- Mastodon: [https://mastodon.social/@lucas3d](https://mastodon.social/@lucas3d)
- Website: [https://www.lucasjanin.com](https://www.lucasjanin.com)
- GitHub: [github.com/lucasjanin](https://github.com/lucasjanin)
- LinkedIn: [linkedin.com/in/lucasjanin](https://linkedin.com/in/lucasjanin)