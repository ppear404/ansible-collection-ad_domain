# microsoft.domain_builder.ad_join

Configure Windows hosts for an existing Active Directory domain and join them as
members, keeping their current hostname and rebooting as required.

The role performs these steps in order:

1. Disable the IPv6 binding on all network adapters.
2. Disable Windows Firewall for the Domain, Public, and Private profiles.
3. Set both DNS servers and the domain search suffix on all adapters.
4. Add a hosts-file entry mapping the domain name to the primary DNS server IP.
5. Join the domain using the configured computer-account OU.

The IPv6 and firewall changes run unconditionally in the normal role entry point;
there are no role variables to turn them off. Account for these changes before
applying the role to an existing host.

## Requirements

- Windows hosts reachable by Ansible with local administrative privileges,
  including after network changes and reboots.
- An existing domain, reachable domain DNS servers, and an existing target OU.
- A domain account permitted to join computers to the domain and target OU.
- The `ansible.windows` and `microsoft.ad` collections on the controller.
- Gathered Windows facts; the role uses `ansible_os_family` and `ansible_hostname`.

There are no automatic role dependencies. Run `ad_ous` first if it is responsible
for creating the target OU.

## Variables

Defaults are defined in [defaults/main.yml](defaults/main.yml).

| Variable | Default | Purpose |
| --- | --- | --- |
| `ad_domain` | `example.com` | Domain to join, DNS search suffix, and hosts-file name. |
| `ad_ou_short` | `example` | First domain component used in the default OU path. |
| `ad_tld` | `com` | Final domain component used in the default OU path. |
| `domain_ou_path` | `OU=myWindows,OU=myServers,DC={{ ad_ou_short }},DC={{ ad_tld }}` | Distinguished name for new computer accounts. |
| `dns_primary` | `192.168.1.10` | Primary DNS server and IP for the hosts-file entry. |
| `dns_secondary` | `192.168.1.11` | Secondary DNS server. |
| `domain_admin` | `myda@example.com` | Account used to join the domain. |
| `vault_domain_admin_password` | Bundled example password | Join-account password; override with a secret. |

For domains with more than two components, supply a complete `domain_ou_path`.
The role passes this value directly to `microsoft.ad.membership`.

## Example

Define `vault_join_password` in an encrypted `vault.yml` file. Configure the
initial Windows connection credentials separately in inventory.

```yaml
- name: Join Windows member servers
  hosts: windows_members
  gather_facts: true
  vars_files:
    - vault.yml
  roles:
    - role: microsoft.domain_builder.ad_join
      ad_domain: corp.example.com
      domain_ou_path: OU=myWindows,OU=myServers,DC=corp,DC=example,DC=com
      dns_primary: 192.168.1.10
      dns_secondary: 192.168.1.11
      domain_admin: join-account@corp.example.com
      vault_domain_admin_password: "{{ vault_join_password }}"
```
