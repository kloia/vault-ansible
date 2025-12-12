# vault-ansible

Ansible role and templates to deploy HashiCorp Vault (raft storage) with TLS. Templates are parameterized from inventory variables; supports an enterprise mode that adds a license path.

## Requirements
- Ansible (>= 2.9)
- Inventory describing `server` group and common variables (see example)
- SSH access to targets (keys or other auth)
- If you want to install enterprise, you need to put vaultpoc.hclic file under ansible role file directory.
- You should create self-signed certificates to secure comminication netween raft nodes. Please look [CERT.MD](CERT.MD)

## Inventory example
```ini
; inventory/vault.ini
[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/dev-key

[server]
ip-172-31-35-96.eu-central-1.compute.internal ansible_host=1.2.3.4
ip-172-31-37-53.eu-central-1.compute.internal ansible_host=1.2.3.5
ip-172-31-44-113.eu-central-1.compute.internal ansible_host=1.2.3.6
```

## Key variables
- `enterprise` (bool) — when true, the template will include `license_path`.
- `license_path` (string, optional) — path to enterprise license file (default in templates: `/etc/vault.d/vaultpoc.hclic`).
- Standard Ansible inventory variables such as `ansible_host`, `ansible_user`, `ansible_ssh_private_key_file` are used by the templates.

## Template behavior (vault.hcl.j2)
- Uses hostvars to render addresses from `ansible_host`:
  - cluster_addr: https://{{ hostvars[inventory_hostname]['ansible_host'] }}:8201
  - api_addr: https://{{ hostvars[inventory_hostname]['ansible_host'] }}:8200
  - node_id: {{ hostvars[inventory_hostname]['ansible_host'] }}
- Builds `retry_join` blocks for every host in `groups['server']` using each host's `ansible_host`.
- Conditionally adds `license_path` when `enterprise` is true:
```jinja
{% if enterprise %}
license_path = "/etc/vault.d/vaultpoc.hclic"
{% endif %}
```

## Usage
Run your playbook pointing to the inventory:
```bash
ansible-playbook -i inventory/vault.ini playbook.yml
```
Ensure TLS certificates and (if enterprise) license file are provisioned on targets at the paths expected by the template (`/opt/vault/tls/*`, `/etc/vault.d/*`).

## TLS & License notes
- TLS certs expected under `/opt/vault/tls/` in the template (`vault.crt`, `vault.key`, `myvault-ca.crt`).
- Enterprise license only included when `enterprise` variable is set to true in inventory/group vars.

## Contributing
Open issues or PRs with minimal reproductions and tests for template changes.

## License
Project-level license as appropriate. Templates include HashiCorp header comments where present.