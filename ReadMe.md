# Bootstrap
Ansible linux system bootstrap role

**Note:** this role currently only supports debian-based systems

**Tested:**
* Debian 11

## Functionality

* Package installation
  * Ansible dependencies (_minimal_)
  * Administrative tools
  * Virtual machine guest-tools (_vm only_)
* Configuration
  * Hostname & local dns resolution
  * Default opt-in:
    * OpenSSH server
    * Users => using [THIS](https://github.com/ansibleguy/base-users) role
    * UFW => using [THIS](https://github.com/ansibleguy/base-ufw) role
    * Network => using [THIS](https://github.com/ansibleguy/base-network) role
  * Default opt-out:
    * auto-updates


**Note:** Most of this functionality can be opted in or out using the main defaults file and variables!


## Setup
To use the full functionality of this role you must first install its dependencies:

```
ansible-galaxy install -r requirements.yml
```

## Usage

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
```
