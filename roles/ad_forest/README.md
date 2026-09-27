# microsoft.domain_builder.ad_forest

Create the root domain of a new Active Directory forest on a Windows Server,
install DNS, and reboot as required. The role then ensures the domain's primary
DNS forward lookup zone exists with domain replication and secure dynamic updates.

## Requirements

- A Windows Server suitable for hosting Active Directory Domain Services, reachable
  by Ansible with administrative privileges, including after a reboot.
- The `microsoft.ad` and `ansible.windows` collections installed on the controller.
- Gathered Windows facts (`gather_facts: true`); the role checks `ansible_os_family`.

There are no automatic role dependencies.

## Variables

Defaults are defined in [defaults/main.yml](defaults/main.yml).

| Variable | Default | Purpose |
| --- | --- | --- |
| `dns_domain` | `example.com` | Fully qualified name of the forest root domain and DNS zone. |
| `safemodepass` | Bundled example password | Directory Services Restore Mode password; override with a secret. |
| `domain_admin` | `myda@example.com` | Used only by the separate `promote_dc.yml` task file. |
| `vault_domain_admin_password` | Bundled example password | Used only by the separate `promote_dc.yml` task file. |

The normal role entry point does not include [tasks/promote_dc.yml](tasks/promote_dc.yml),
so running this role does not promote additional domain controllers. The two
administrator variables are unused during normal forest creation.

## Example

Define `vault_ad_safe_mode_password` in an encrypted variables file loaded by the
playbook. Configure connection credentials separately in inventory.

```yaml
- name: Create the forest root domain
  hosts: forest_root
  gather_facts: true
  vars_files:
    - vault.yml
  roles:
    - role: microsoft.domain_builder.ad_forest
      dns_domain: corp.example.com
      safemodepass: "{{ vault_ad_safe_mode_password }}"
```
