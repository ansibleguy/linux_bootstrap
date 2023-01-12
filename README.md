# Ansible Role - Linux Bootstrap
Ansible Role to bootstrap linux servers.

It runs some basic setup tasks to bring a cleanly installed linux server up to the needed standards for further usage.

[![Molecule Test Status](https://badges.ansibleguy.net/linux_bootstrap.molecule.svg)](https://github.com/ansibleguy/_meta_cicd/blob/latest/templates/usr/local/bin/cicd/molecule.sh.j2)
[![YamlLint Test Status](https://badges.ansibleguy.net/linux_bootstrap.yamllint.svg)](https://github.com/ansibleguy/_meta_cicd/blob/latest/templates/usr/local/bin/cicd/yamllint.sh.j2)
[![Ansible-Lint Test Status](https://badges.ansibleguy.net/linux_bootstrap.ansiblelint.svg)](https://github.com/ansibleguy/_meta_cicd/blob/latest/templates/usr/local/bin/cicd/ansiblelint.sh.j2)
[![Ansible Galaxy](https://img.shields.io/ansible/role/56761)](https://galaxy.ansible.com/ansibleguy/linux_bootstrap)
[![Ansible Galaxy Downloads](https://img.shields.io/badge/dynamic/json?color=blueviolet&label=Galaxy%20Downloads&query=%24.download_count&url=https%3A%2F%2Fgalaxy.ansible.com%2Fapi%2Fv1%2Froles%2F56761%2F%3Fformat%3Djson)](https://galaxy.ansible.com/ansibleguy/linux_bootstrap)

**Tested:**
* Debian 11

## Install

```bash
ansible-galaxy install ansibleguy.linux_bootstrap

# or to custom role-path
ansible-galaxy install ansibleguy.linux_bootstrap --roles-path ./roles

# install dependencies
ansible-galaxy install -r requirements.yml
```

## Functionality

* **Package installation**
  * Ansible dependencies (_minimal_)
  * Administrative tools
  * Virtual machine guest-tools (_vmware/kvm_)
  * lightweight administrative tools


* **Default opt-in**:
  * OpenSSH server
  * Users/Groups => using [THIS](https://github.com/ansibleguy/linux_users) role


* **Default opt-out**:
  * Auto-updates
  * UFW => using [THIS](https://github.com/ansibleguy/linux_ufw) role
  * Network(-interfaces) => using [THIS](https://github.com/ansibleguy/linux_networking) role


## Info

* **Note:** Most of the role's functionality can be opted in or out.

  For all available options - see the default-config located in the main defaults-file!



* **Note:** this role currently only supports debian-based systems


* **Warning:** Not every setting/variable you provide will be checked for validity. Bad config might break the role!


* **Info:** Prerequisites:

  1. You must be able to connect via ssh with a user that has root privileges.
  The easiest way to do this - is to set 'PermitRootLogin' to 'yes' temporarily and restart the sshd service.

  2. Connect to the server one time using ssh to mark the host-key as known.

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

I've not yet found a solution for reloading the 'meta-variables' (_like the targets ip-address, ssh-port and ssh-credentials_) so the bootstrapping can be done in one run. See also: [Issue](https://github.com/ansibleguy/linux_bootstrap/issues/1)

Therefor the bootstrapping got 'part'-flags as shown in the example below. 

Run the playbook:
```bash
# prerequisites:
#   1. you must be able to connect via ssh with a user that has root privileges
#     the easiest way to do this - is to set 'PermitRootLogin' to 'yes' temporarily and restart the sshd service
#   2. connect to the server one time using ssh to mark the host-key as known

# 1. connecting the first time using root, the default ssh-port and currently active ip
#   this part will deploy: basics, auto-update, users & groups, ssh- and ufw-config
#   NOTE: you might need to add the '--ask-vault-pass' flag if you're using ansible-vault to secure your user-passwords

#   example using root
init_user="root"
init_port=22
init_ip="192.168.0.1"
ansible-playbook --ask-pass -D -i inventory/hosts.yml playbook.yml -e ansible_port="$init_port" -e ansible_user="$init_user" -e ansible_host="$init_ip" -e part=1

#   example using other privileged user
ansible-playbook --ask-become-pass -D -i inventory/hosts.yml playbook.yml -e ansible_port="$init_port" -e ansible_user="$init_user" -e ansible_host="$init_ip" -e part=1

# 2. re-run to deploy the network config
#   NOTE: if the ip-address changes - the network task will show an error
#   example using a privileged user
ansible-playbook --ask-become-pass -D -i inventory/hosts.yml playbook.yml -e ansible_host="$init_ip" --ask-vault-pass -e part=2

# after this setup you can re-run the bootstrapping as often as you want/need to update its config
#   NOTE: you might need to add the '--ask-vault-pass' flag if you're using ansible-vault to secure your user-passwords
ansible-playbook -K -D -i inventory/hosts.yml playbook.yml
```

There are also some useful **tags** available:
* base
* interfaces
* routing
* auth
* update
* ufw
* ssh
* part1
* part2
