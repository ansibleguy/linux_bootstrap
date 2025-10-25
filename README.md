# Ansible Role - Linux Bootstrap
Ansible Role to bootstrap linux servers.

It runs some basic setup tasks to bring a cleanly installed linux server up to the needed standards for further usage.

[![Lint](https://github.com/O-X-L/ansible-role-linux-bootstrap/actions/workflows/lint.yml/badge.svg)](https://github.com/O-X-L/ansible-role-linux-bootstrap/actions/workflows/lint.yml)
[![Ansible Galaxy](https://badges.oss.oxl.app/galaxy.badge.svg)](https://galaxy.ansible.com/ui/standalone/roles/oxlorg/linux_bootstrap)

**Molecule Integration-Tests**:

* Status: [![Molecule Test Status](https://badges.oss.oxl.app/linux_bootstrap.molecule.svg)](https://github.com/O-X-L/ansible-role-oxl-cicd/blob/latest/templates/usr/local/bin/cicd/molecule.sh.j2) |
[![Functional-Tests](https://github.com/O-X-L/ansible-role-linux-bootstrap/actions/workflows/integration_test_result.yml/badge.svg)](https://github.com/O-X-L/ansible-role-linux-bootstrap/actions/workflows/integration_test_result.yml)
* Logs: [API](https://ci.oss.oxl.app/api/job/ansible-test-molecule-linux_bootstrap/logs?token=2b7bba30-9a37-4b57-be8a-99e23016ce70&lines=1000) | [Short](https://badges.oss.oxl.app/log/molecule_linux_bootstrap_test_short.log) | [Full](https://badges.oss.oxl.app/log/molecule_linux_bootstrap_test.log)

Internal CI: [Tester Role](https://github.com/O-X-L/ansible-role-oxl-cicd) | [Jobs API](https://github.com/O-X-L/github-self-hosted-jobs-systemd)

**Tested:**
* Debian 11
* Debian 12

----

## Install

```bash
# latest
ansible-galaxy role install git+https://github.com/O-X-L/ansible-role-linux-bootstrap

# from galaxy
ansible-galaxy install oxlorg.linux_bootstrap

# or to custom role-path
ansible-galaxy install oxlorg.linux_bootstrap --roles-path ./roles

# install dependencies
ansible-galaxy install -r requirements.yml
python3 -m pip install -r requirements.txt
```

----

## Advertisement

* Need **professional support** using Ansible or Linux? Contact us:

  E-Mail: [contact@oxl.at](mailto:contact@oxl.at)

  Tel: [+43 3115 40 900 0](tel:+433115409000)

  Web: [EN](https://www.o-x-l.com) | [DE](https://www.oxl.at)

  Language: German or English

* You want a simple **Ansible GUI**?

  Check-out this [Ansible WebUI](https://github.com/O-X-L/ansible-webui)

----

## Usage

### Config

Define the ssh/update/user/group/network/ufw config as needed.

```yaml
bootstrap:
  configure_network: true
  configure_firewall: true
  configure_users: true
  install_tools: true
  
  host_fqdn: 'host.bootstrap.template.oxl.at'  # optional
  
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
  users:  # more info: https://github.com/O-X-L/ansible-role-linux-users
    guy:
      comment: 'AnsibleGuy'
      ssh_pub: 'ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKkIlii1iJM240yPSPS5WhrdQwGFa7BTJZ59ia40wgVWjjg1JlTtr9K2W66fNb2zNO7tLkaNzPddMEsov2bJAno= contact@oxl.at'
  
  groups:
    ag_users:
      members: []
    ag_admins:
      members: ['guy']
      member_of: ['ag_users']

network:  # more info: https://github.com/O-X-L/ansible-role-linux-networking
  interfaces:
    ens192:
      address: '192.168.142.90/24'
      gateway: '192.168.142.1'
```

### Execution

I've not yet found a solution for reloading the 'meta-variables' (_like the targets ip-address, ssh-port and ssh-credentials_) so the bootstrapping can be done in one run. See also: [Issue](https://github.com/O-X-L/ansible-role-linux-bootstrap/issues/1)

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

----

## Functionality

* **Package installation**
  * Ansible dependencies (_minimal_)
  * Administrative tools
  * Virtual machine guest-tools (_vmware/kvm_)
  * lightweight administrative tools


* **Default opt-in**:
  * OpenSSH server
  * Users/Groups => using [THIS](https://github.com/O-X-L/ansible-role-linux-users) role


* **Default opt-out**:
  * Auto-updates
  * Network(-interfaces) => using [THIS](https://github.com/O-X-L/ansible-role-linux-networking) role


## Info

* **Note:** Most of the role's functionality can be opted in or out.

  For all available options - see the default-config located in [the main defaults-file](https://github.com/O-X-L/ansible-role-linux-bootstrap/blob/latest/defaults/main/1_main.yml)!



* **Note:** this role currently only supports debian-based systems


* **Warning:** Not every setting/variable you provide will be checked for validity. Bad config might break the role!


* **Info:** Prerequisites:

  1. You must be able to connect via ssh with a user that has root privileges.
  The easiest way to do this - is to set 'PermitRootLogin' to 'yes' temporarily and restart the sshd service.

  2. Connect to the server one time using ssh to mark the host-key as known.
