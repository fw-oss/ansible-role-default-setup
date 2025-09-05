# Ansible Role default-setup

Executes a basic setup on a blank server

## Requirements

Ansible Version 2.9

## Dependencies

[Community General Collection](https://docs.ansible.com/ansible/latest/collections/community/general/index.html) (comes with `ansible`, but not with `ansible-core`) for `community.general.timezone`, `community.general.ufw ` 
[Ansible Posix Collection](https://docs.ansible.com/ansible/latest/collections/ansible/posix/index.html) (comes with `ansible`, but not with `ansible-core`) for `ansible.posix.mount`


## Example Playbook

```yaml
---
- hosts: all
  vars:
    domain: intranet.example.com
    auto_upgrade_mail: sysadmin@example.com
    auto_upgrade_mail_trigger: on-change
    mail_configuration: smarthost
    smtp_server: contoso-com.mail.protection.outlook.com
    ssh_login_notification_mail: duke@company.example it-support@example.com
    sender_email_domain: contoso.com
    ssh_login_notification_webhooks:
      - name: mattermost
        url: chat.example.com/12345
        options: >-
          -i -X POST --data-urlencode 'payload={"text": "SSH-Login\nUser:
          '"$PAM_USER"'\nRemote Host: '"$PAM_RHOST"'\nService:
          '"$PAM_SERVICE"'\nTTY: '"$PAM_TTY"'\nDate: '"$(date)"'\nHost:
          '"$(hostname -f)"'\nServer: '"$(uname -a)"'", "username":
          "SSH Login PAM Webhook"}'
      - name: discord
        url: https://dicord.com/api/webhooks
        options: >-
          -X POST -H 'Content-Type: application/json' -d "{ \"content\":
          \"$DISCORDUSER: User \`$PAM_USER\` logged in to \`$HOSTNAME\`
          (remote host: $PAM_RHOST).\" }"
      - name: fax
        state: absent
  roles:
    - role: fw_oss.default_setup
      become: true
```

```yaml
---
- hosts: all
  vars:
    ssh_permit_root_login: true
    ssh_allow_users: "*@192.168.0.* peter@192.168.255.1"
    inotify_max_queued_events: 16384
    inotify_max_user_instances: 32768
    inotify_max_user_watches: 8192
    max_map_count: 524288
    swappiness: 10
    net_core_somaxconn: 65535
  roles:
    - role: fw_oss.default_setup
      become: true
```

## Details

- installs packages
- sets hostname
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
- configures sudo:
    - creates group "admin" with passwordless full privileges
    - sets mail adress for unauth. sudo attempts (excl. -l, -v)  
- changes root prompt
- sets timezone
- mail on login
- sets swap:
    - check for swap
    - check for zfs
    - if both fail: create swapfile etc.
- some kernel stuff:
    - swapiness

## Role Variables

```yml

domain: "example.com"
ca_certificates:
  - "cert 1"
  - "cert 2"
smtp_server: ""
mail_configuration: 'smarthost' or 'standalone'

################
# Auto Update  #
################

auto_upgrade_enable: 1
auto_upgrade_update_package_list: 1
auto_upgrade_unattended_upgrade: 1
auto_upgrade_autoclean_interval: 7
auto_upgrade_mail: update@example.com

####################
# System Settings  #
####################

inotify_max_queued_events: 16384
inotify_max_user_instances: 32768
inotify_max_user_watches: 8192
max_map_count: 524288
swappiness: 10
swap_file_path: "/swapfile"
swap_size_mb: 4096
net_core_somaxconn: 65535
timezone_default: Europe/Berlin

########
# SSH  #
########

ssh_permit_root_login: "no"
ssh_client_alive_interval: 300
ssh_client_alive_count_max: 5
ssh_port: 22
ssh_password_authentication: "no"
ssh_x11_forwarding: "no"
ssh_max_auth_tries: 5
ssh_allow_tcp_forwarding: "no"
ssh_allow_agent_forwarding: "no"
ssh_authorized_keys_file: ".ssh/authorized_keys"
ssh_pubkey_authentication: "yes"
ssh_challenge_response_authentication: "no"
ssh_login_notification_mail:

########
# APT  #
########

apt_cache_valid_time: 3600
apt_install_packages:
  - nload
  - htop
  - iotop
  - git
  - ncdu
  - tmux
  - ncftp
  - mtr
  - curl
  - wget
  - nano
  - rsync
  - zip
  - unzip
  - dnsutils
  - sudo
  - apt-dater-host
  - systemd-timesyncd # available Debian 10 / Ubuntu 20.04 upwards
  - tzdata # for timezone
  - needrestart
  - neofetch
  - pigz
  - mc
  - nmap
  - net-tools
  - openssh-client
  - python3-pip
  - libffi-dev
  - libssl-dev
apt_remove_packages:
  - ntp
apt_setup_unattended_upgrades: true

```

### ToDo:
- set shell
- unattended upgrades
- activate PAM
- updates certificates
- periodic file system trim


## License

MIT

## Author Information

FW-OSS, 2024
