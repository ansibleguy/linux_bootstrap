# Ansible Role - Linux Bootstrap
Ansible Role to bootstrap linux servers.

It runs some basic setup tasks to bring a cleanly installed linux server up to the needed standards for further usage.


**Tested:**
* Debian 11

## Functionality

* **Package installation**
  * Ansible dependencies (_minimal_)
  * Administrative tools
  * Virtual machine guest-tools (_vm only_)
  * lightweight administrative tools


* **Default opt-in**:
  * OpenSSH server
  * Users/Groups => using [THIS](https://github.com/ansibleguy/linux_users) role


* **Default opt-out**:
  * Auto-updates
  * UFW => using [THIS](https://github.com/ansibleguy/linux_ufw) role
  * Network(-interfaces) => using [THIS](https://github.com/ansibleguy/linux_networking) role


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
bootstrap:
  configure_network: true
  configure_firewall: true
  configure_users: true
  install_tools: true
  
  host_fqdn: 'host.bootstrap.template.ansibleguy.net'  # optional
  
  ssh:
    configure: true
    port: 10022
    auto_pwd: false
    # auth_multi: true  # if you want to enforce pwd & pubkey combined for ssh-authentication
    msg: true  # show pre- and post-login banners
    welcome_msg:
      - 'Welcome to the secret server!'

  auto_update:
    enable: true
    exclude_kernel: true
    exclusions: ['haproxy']
    logging_verbose: true

system_auth:
  users:  # more info: https://github.com/ansibleguy/linux_users
    guy:
      comment: 'AnsibleGuy'
      ssh_pub: 'ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKkIlii1iJM240yPSPS5WhrdQwGFa7BTJZ59ia40wgVWjjg1JlTtr9K2W66fNb2zNO7tLkaNzPddMEsov2bJAno= guy@ansibleguy.net'
  
  groups:
    ag_users:
      members: []
    ag_admins:
      members: ['guy']
      member_of: ['ag_users']

network:  # more info: https://github.com/ansibleguy/linux_networking
  interfaces:
    ens192:
      address: '192.168.142.90/24'
      gateway: '192.168.142.1'

ufw_rules:  # more info: https://github.com/ansibleguy/linux_ufw
  ssh:
    port: 10022
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
# connecting the first time using root and the default ssh-port:
ansible-playbook -k -D -i inventory/hosts.yml playbook.yml -e ansible_port=22 -e ansible_user=root --ask-vault-pass

# or (re-)run it with a privileged user:
ansible-playbook -K -D -i inventory/hosts.yml playbook.yml --ask-vault-pass
```

There are also some useful **tags** available:
* base
* interfaces
* routing
* auth
* update
* ufw
* ssh
