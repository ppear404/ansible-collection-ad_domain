# microsoft.domain_builder

An Ansible collection for creating an Active Directory forest, organizing its
objects, provisioning accounts, and joining Windows hosts to the domain.

## Roles

| Role | Purpose |
| --- | --- |
| [ad_forest](roles/ad_forest/README.md) | Create a forest root domain with DNS, reboot as needed, and configure its forward lookup zone. |
| [ad_ous](roles/ad_ous/README.md) | Create a configurable hierarchy of organizational units. |
| [ad_users](roles/ad_users/README.md) | Create enabled administrator and general-user accounts in designated OUs and add their group memberships. |
| [ad_join](roles/ad_join/README.md) | Configure Windows networking and join hosts to an existing domain, rebooting as needed. |

`ad_join` disables IPv6 bindings and all Windows Firewall profiles, changes DNS
settings on all adapters, and adds a domain hosts-file entry before joining.
These steps are part of its normal execution, with no variable-based opt-out.
`ad_forest` creates a new forest; its additional-controller promotion task file is
not included by the normal role entry point.

## Requirements and installation

Use an Ansible controller with the `ansible.windows` and `microsoft.ad` collections
and a configured Windows connection with appropriate administrative permissions.
All roles require gathered Windows facts. Forest creation requires Windows Server;
OU and user management require an existing domain and the Active Directory
PowerShell module on the managed host. Connections must survive or reconnect after
network changes and reboots.

The collection currently declares no collection dependencies in `galaxy.yml`, so
install its module dependencies explicitly. From this repository directory:

```bash
ansible-galaxy collection install ansible.windows microsoft.ad
ansible-galaxy collection build
ansible-galaxy collection install ./microsoft-domain_builder-1.0.0.tar.gz
```

The artifact name reflects the version in [galaxy.yml](galaxy.yml). No supported
minimum Ansible version is currently declared in [meta/runtime.yml](meta/runtime.yml).

## Configuration

Set Windows connection details and credentials in inventory or group variables.
Replace the bundled example passwords and account lists before use; keep secrets
in Ansible Vault and load them into the playbook. A variable name starting with
`vault_` does not encrypt its value automatically.

The forest role uses `dns_domain`, while user provisioning and domain joining use
`ad_domain`; set both consistently. Default OU paths use `ad_ou_short` and `ad_tld`
for a two-component domain such as `example.com`. For longer domain names, provide
complete distinguished names in the OU list and the user/computer OU variables.
See each role README for its variables and examples.

## Example workflow

Run forest creation first, then create OUs and accounts using a domain-authorized
connection, and finally join member hosts. Roles do not invoke one another as
dependencies. The following playbook uses the default OU hierarchy for
`example.com`:

```yaml
- name: Create the forest
  hosts: forest_root
  gather_facts: true
  vars_files:
    - vault.yml
  roles:
    - role: microsoft.domain_builder.ad_forest
      dns_domain: example.com
      safemodepass: "{{ vault_ad_safe_mode_password }}"

- name: Provision directory objects
  hosts: directory_management
  gather_facts: true
  vars_files:
    - vault.yml
  roles:
    - role: microsoft.domain_builder.ad_ous
      ad_ou_short: example
      ad_tld: com
    - role: microsoft.domain_builder.ad_users
      ad_domain: example.com
      ad_ou_short: example
      ad_tld: com
      domain_admins: "{{ vault_domain_admins }}"
      domain_users: "{{ vault_domain_users }}"

- name: Join member servers
  hosts: windows_members
  gather_facts: true
  vars_files:
    - vault.yml
  roles:
    - role: microsoft.domain_builder.ad_join
      ad_domain: example.com
      domain_ou_path: OU=myWindows,OU=myServers,DC=example,DC=com
      dns_primary: 192.168.1.10
      dns_secondary: 192.168.1.11
      domain_admin: "{{ vault_join_username }}"
      vault_domain_admin_password: "{{ vault_join_password }}"
```

Define the three inventory groups and load an encrypted `vault.yml` containing the
referenced secrets. `vault_domain_admins` and `vault_domain_users` must each be a
list of objects with `user_name` and `user_pwd`. Use a single domain controller or
management host in `directory_management` to avoid managing the same directory
objects concurrently. If that host is also the newly promoted forest root, configure
its connection for domain authentication for the provisioning play.

## License

The collection metadata declares `GPL-2.0-or-later` in [galaxy.yml](galaxy.yml).
