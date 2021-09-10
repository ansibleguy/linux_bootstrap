# Bootstrap
Ansible linux system bootstrap role

**Note:** this role currently only supports debian-based systems

## Functionality

This ansible role will configure:
* Package installation
  * Ansible essentials
  * administrative tools
  * virtual machine guest-tools (_vm only_)
* Configuration
  * hostname & local dns resolution
  * default opt-in:
    * OpenSSH server
    * Users => using [THIS](https://github.com/ansibleguy/base-users) role
    * UFW => using [THIS](https://github.com/ansibleguy/base-ufw) role
  * default opt-out:
    * auto-updates


**Note:** Most of this functionality can be opted in or out using the main defaults file and variables!


## Setup
To use the full functionality of this role you must first install its dependencies:

```
ansible-galaxy install -r requirements.yml
```
