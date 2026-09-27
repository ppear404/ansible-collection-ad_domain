# microsoft.domain_builder.ad_ous

Ensure a configured list of Active Directory organizational units exists. OUs are
created in list order using each entry's name and parent distinguished name.
Removing an entry from the list does not delete its OU.

## Requirements

- An existing Active Directory domain and a Windows management host with the
  Active Directory PowerShell module, such as a domain controller.
- An Ansible connection account permitted to create OUs in the specified paths.
  The role uses the connection's security context; it defines no separate domain
  credential variables.
- The `microsoft.ad` collection installed on the controller.
- Gathered Windows facts (`gather_facts: true`).

There are no automatic role dependencies. Create the domain before running this
role, and put parent OUs before their children in the list.

## Variables

Defaults are defined in [defaults/main.yml](defaults/main.yml).

| Variable | Default | Purpose |
| --- | --- | --- |
| `ad_ou_short` | `example` | First domain component used in default parent paths. |
| `ad_tld` | `com` | Final domain component used in default parent paths. |
| `ad_organizational_units` | The hierarchy below | List of objects with required `name` and `path` fields. `path` is the parent DN, not the new OU's full DN. |

The default hierarchy under `DC=example,DC=com` is:

```text
myUsers
  myPrivilegedUsers
  myGeneralUsers
myGroups
myComputers
myServiceAccounts
myServers
  myLinux
  myWindows
myWorkstations
  myFinance
  myOperations
```

Override `ad_organizational_units` to replace the default list or supply full DNs
for a domain with more than two components.

## Example

```yaml
- name: Create organizational units
  hosts: domain_controllers
  gather_facts: true
  roles:
    - role: microsoft.domain_builder.ad_ous
      ad_organizational_units:
        - name: myServers
          path: DC=corp,DC=example,DC=com
        - name: myWindows
          path: OU=myServers,DC=corp,DC=example,DC=com
```
