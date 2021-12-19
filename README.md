# Linux Bootstrap
Ansible Role to bootstrap linux servers.

It runs some basic setup tasks to bring a cleanly installed linux server up to the needed standards for further usage.


**Tested:**
* Debian 11

## Functionality

* **Package installation**
  * Ansible dependencies (_minimal_)
  * Administrative tools
  * Virtual machine guest-tools (_vm only_)


* **Configuration**
  * Hostname & local dns resolution
  * Default opt-in:
    * OpenSSH server
    * Users/Groups => using [THIS](https://github.com/ansibleguy/linux_users) role
    * UFW => using [THIS](https://github.com/ansibleguy/infra_ufw) role
    * Network => using [THIS](https://github.com/ansibleguy/linux_networking) role


  * **Default opt-out**:
    * auto-updates


## Info

* **Note:** Most of this functionality can be opted in or out using the main defaults file and variables!


* **Note:** this role currently only supports debian-based systems


* **Warning:** Not every setting/variable you provide will be checked for validity. Bad config might break the role!


## Requirements

* Community collection and sub-roles: ```ansible-galaxy install -r requirements.yml```


## Usage

### Config

Define the ssh/update/user/group/network/ufw config as needed.

```yaml
ssh_config:
  port: 10022
  auto_pwd: false
  # auth_multi: true  # if you want to enforce pwd & pubkey combined for ssh-authentication
  msg: true  # show login pre- and post-login banner
  welcome_msg:
    - 'Welcome! :D'
    - "And don't forget => 'rm -rf / --no-preserve-root' is bad practise"

configure_auto_update: true
auto_update_config:
  exclude_kernel: true
  exclusions: ['haproxy']
  logging_verbose: true

users:  # more info: https://github.com/ansibleguy/linux_users
  guy:
    comment: 'AnsibleGuy'
    ssh_pub: 'ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKkIlii1iJM240yPSPS5WhrdQwGFa7BTJZ59ia40wgVWjjg1JlTtr9K2W66fNb2zNO7tLkaNzPddMEsov2bJAno= guy@ansibleguy.net'

user_groups:
  ag_users:
    members: []
  ag_admins:
    members: ['guy']
    member_of: ['ag_users']

network_interfaces:  # more info: https://github.com/ansibleguy/linux_networking
  ens192:
    address: '192.168.142.90/24'
    gateway: '192.168.142.1'

ufw_rules:  # more info: https://github.com/ansibleguy/infra_ufw
  ssh:
    port: 22
    proto: 'tcp'
    log: true
    rule: 'limit'
  webServer:
    port: 80,443
    proto: 'tcp'
```

### Execution

Run the playbook:
```bash
# first time
ansible-playbook -k -D -i inventory/hosts.yml playbook.yml -e ansible_port=22 -e ansible_user=root --ask-vault-pass```

# subsequent runs
ansible-playbook -K -D -i inventory/hosts.yml playbook.yml --ask-vault-pass
```