# microsoft.domain_builder.ad_users

Create and enable Active Directory administrator and general-user accounts in
separate OUs. Administrator accounts are added to `Domain Admins`; general-user
accounts are added to `Domain Users`. Each account's name and SAM account name
come from `user_name`, and its UPN is `user_name@ad_domain`.

The role ensures listed accounts are present. Removing an account from a list or
disabling a creation branch does not delete existing accounts or revoke membership.

## Requirements

- An existing Active Directory domain and the destination OUs.
- A Windows management host with the Active Directory PowerShell module, such as
  a domain controller, and an Ansible connection account permitted to manage the
  users and group memberships. The role uses the connection's security context.
- The `microsoft.ad` collection installed on the controller.
- Gathered Windows facts (`gather_facts: true`).

There are no automatic role dependencies. Run `ad_ous` first when using its default
user OU hierarchy.

## Variables

Defaults are defined in [defaults/main.yml](defaults/main.yml).

| Variable | Default | Purpose |
| --- | --- | --- |
| `add_admins` | `true` | Run the administrator-account tasks. |
| `add_users` | `true` | Run the general-user tasks. |
| `ad_domain` | `example.com` | UPN suffix for all listed accounts. |
| `ad_ou_short` | `example` | First domain component used in default OU paths. |
| `ad_tld` | `com` | Final domain component used in default OU paths. |
| `priv_users_ou` | `OU=myPrivilegedUsers,OU=myUsers,DC={{ ad_ou_short }},DC={{ ad_tld }}` | Destination OU for administrator accounts. |
| `users_ou` | `OU=myGeneralUsers,OU=myUsers,DC={{ ad_ou_short }},DC={{ ad_tld }}` | Destination OU for general users. |
| `domain_admins` | One example account, `myda` | List of objects with required `user_name` and `user_pwd` fields. |
| `domain_users` | One example account, `mydu` | List of objects with required `user_name` and `user_pwd` fields. |

Both account lists include example passwords. Replace them with your own lists
and encrypted secrets before running the role. Both user-creation tasks use
`no_log: true`. For domains with more than two components, override both OU paths
as needed; changing `ad_domain` does not change those paths.

## Example

Define `vault_domain_admins` and `vault_domain_users` in encrypted `vault.yml`, each
as a list of `user_name` and `user_pwd` objects. Use empty lists or set the matching
`add_*` flag to `false` for account types you do not want to manage.

```yaml
- name: Create domain accounts
  hosts: domain_controllers
  gather_facts: true
  vars_files:
    - vault.yml
  roles:
    - role: microsoft.domain_builder.ad_users
      ad_domain: corp.example.com
      priv_users_ou: OU=myPrivilegedUsers,OU=myUsers,DC=corp,DC=example,DC=com
      users_ou: OU=myGeneralUsers,OU=myUsers,DC=corp,DC=example,DC=com
      domain_admins: "{{ vault_domain_admins }}"
      domain_users: "{{ vault_domain_users }}"
```
