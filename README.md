# Ansible Role default-setup

Executes a basic setup on a blank server

## Example Playbook

```yaml
---
- hosts: all
  vars:
  roles:
    - role: fw_oss.default_setup
      become: true
```

## Details

- installs packages
- sets hostname
- set shell (TODO)
- unattended upgrades (TODO)
- hardenes ssh:
    - install, config and enable fail2ban
    - disable ufw
    - permit/disallow Root login
    - set/disable ClientAliveInterval and ClientAliveCountMax
    - set port (and allow in ufw)
    - permit/disallow Password auth
    - permit/disallow X11 Forwarding
    - set/disable MaxAuthTries
    - permit/disallow TCP Forwarding
    - permit/disallow Agent Forwarding
    - define file for authorized keys
    - PubkeyAuthentication
    - ChallengeResponseAuthentication
    - (TODO activate PAM?)
- configures sudo:
    - creates group "admin" with passwordless full privileges
    - sets mail adress for unauth. sudo attempts (excl. -l, -v)
    - enables insults on wrong passwd (why???)
- changes root prompt (TODO)
- sets timezone
- updates certificates (TODO)
- mail:
    - (TODO)
- sets swap:
    - check for swap
    - check for zfs
    - if both fail: create swapfile etc.
- periodic file system trim (TODO)
- mail on login
    - (TODO)
- some kernel stuff:
    - (TODO)
    - swapiness

## Dependencies

[Community General Collection](https://docs.ansible.com/ansible/latest/collections/community/general/index.html) (comes with `ansible`, but not with `ansible-core`) for community.general.timezone, community.general.ufw  
[Ansible Posix Collection](https://docs.ansible.com/ansible/latest/collections/ansible/posix/index.html) (comes with `ansible`, but not with `ansible-core`) for ansible.posix.mount


## License

MIT

## Author Information

FW-OSS, 2024

## Variables

Required:

ssh_login_notification_mail: 
auto_upgrade_mail:
domain:

TODO:

mail_configuration: 'smarthost' or 'standalone'