# Ansible Role: Default Setup

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
    sudo_notification_mail: duke@company.example
    ssh_login_notification_mail: duke@company.example it-support@example.com
    sender_email_domain: contoso.com
    ssh_login_notification_webhooks:
      - name: mattermost
        url: chat.example.com/12345
        options: >-
          -i -X POST -m 10 --data-urlencode 'payload={"text": "SSH-Login\nUser:
          '"$PAM_USER"'\nRemote Host: '"$PAM_RHOST"'\nService:
          '"$PAM_SERVICE"'\nTTY: '"$PAM_TTY"'\nDate: '"$(date)"'\nHost:
          '"$(hostname -f)"'\nServer: '"$(uname -a)"'", "username":
          "SSH Login PAM Webhook"}'
      - name: discord
        url: https://dicord.com/api/webhooks
        options: >-
          -X POST -m 10 --retry 5 -H 'Content-Type: application/json'
          -d "{ \"content\": \"$DISCORDUSER: User \`$PAM_USER\` logged in to \`$HOSTNAME\`
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

### Upgrades and installs packages, reboots if necessary

```yaml
apt_cache_valid_time: 3600
apt_install_packages:
  # System configuration:
  - sudo
  - apt-dater-host  # to remotely manage updates via apt-dater
  - tzdata  # for timezones
  - systemd-timesyncd # available Debian 10 / Ubuntu 20.04 upwards  client only ntp
  - needrestart  # checks which daemons need to be restarted after library upgrades
  - net-tools  # networking toolkit
  - openssh-client

  # Admin tools
  # debug:
  - nload  # realtime console network usage monitor
  - htop  # like task manager
  - iotop  # Why is the disk churning so much?
  - ncdu  # disk usage
  - mtr  # traceroute + ping
  - neofetch  # system info
  - dnsutils  # TODO: "Transitional package" for bind9-~
  # - bind9-dnsutils  # dig, nslookup, nsupdate
  - nmap  # network exploration and security auditing

  # download
  - curl
  - wget
  - rsync
  - ncftp  # ftp client

  # compress
  - zip
  - unzip
  - pigz

  # other tools:
  - tmux  # open multiple persistent terminals
  - mc  # text-mode full-screen file manager
  - git
  - nano

  # Misc
  - python3-pip  # ?
  - libffi-dev  # ?
  - libssl-dev  # ?
apt_remove_packages:
  - ntp  # replaced with systemd-timesyncd
```

### Sets hostname to inventory hostname

### Sets default shell

```yaml
setup_default_shell: /bin/bash
```

### Enables automatic upgrades

```yaml
apt_setup_unattended_upgrades: true

auto_upgrade_enable: 1
auto_upgrade_update_package_list: 1  #  package lists refreshing interval in days. 0 to disable
auto_upgrade_unattended_upgrade: 1  # upgrade interval in days. 0 to disable
auto_upgrade_autoclean_interval: 7  # remove obsolete packages every x days

auto_upgrade_mail:
auto_upgrade_mail_trigger:  # Set this value to one of: "always", "only-on-error" or "on-change"

```

### SSH hardening

configuring `/etc/ssh/sshd_config`

```yaml
# Specifies whether root can log in using ssh.
# Options: yes, prohibit-password, forced-commands-only, or no
ssh_permit_root_login: "no"
# timeout interval in seconds after which if no data has been received from the client,
# sshd will send a message through the encrypted channel to request a response from the client.
ssh_client_alive_interval: 300
# disconnect after this amount of above response requests
ssh_client_alive_count_max: 5
ssh_port: 22
# allow/disallow Password auth
ssh_password_authentication: "no"
# allow/disallow X11 (i.e. GUI) forwarding
ssh_x11_forwarding: "no"
# Specifies the maximum number of authentication attempts per connection, default 6
ssh_max_auth_tries: 5
# allow/disallow tcp forwarding. yes,all | no | local | remote
ssh_allow_tcp_forwarding: "no"
# allow/disallow agent forwarding
ssh_allow_agent_forwarding: "no"
ssh_authorized_keys_file: ".ssh/authorized_keys"
# allow/disallow auth via public key
ssh_pubkey_authentication: "yes"
# allow/disallow keyboard authentication (deprecated)
ssh_challenge_response_authentication: "no"
# allow only specific users from specific locations
# list of USER@HOST, separated by spaces with ? and * as wildcards
# may contain CIDR notation
ssh_allow_users:
```

Enable/disable ufw (a simple firewall)
```yaml
ufw_enabled: "false"

```
This will also set above ssh port as allowed in ufw

Also enables fail2ban

### Makes root shell prompt red

#### Variables

```yaml
domain:  # used to split $(hostname -f)
```

#### Code
```bash
PS1="\[\033[1;41;37m\]\u\[\033[0;41;37m\]@$(hostname -f
      {{ '| sed "s/.' + domain + '//"' if domain is defined }})(\l):\w\[\033[41;00m\]$ "
```

#### Usage Example

hostname = worker02.dmz.company.org,  
`domain: company.org`  
Result (in red):
```bash
root@worker02.dmz(0):/home/user$ █
```

### Sets default timezone

Replacing `ntp` with `systemd-timesyncd`, a client-only ntp implementation.  
See package list above  

```yaml
timezone_default: Europe/Berlin
```
### Downloads CA certificates

```yaml
ca_certificates: []
```

### Configures mail

```yaml
mail_configuration: 'smarthost' or 'standalone'
sender_email_domain:
smtp_server:
```

### Swap

Checks for existing swap and if zfs is used ([zfs got issues]((https://github.com/openzfs/zfs/issues/7734)))  
If those checks fail, creates and activates a swap file

```yaml
swap_file_path: "/swapfile"
swap_size_mb: 4096
```

### Ensures a periodic fstrim run

Checks that service `fstrim.timer` is enabled

### Kernel settings

creates a file at `/etc/sysctl.d/20-ansible-fw-oss-default-setup.conf` if at least one of the following vars are defined

```yaml
# Change these if you know what you're doing or a random app tells you to:
# File system watchers
inotify_max_queued_events:
inotify_max_user_instances:
inotify_max_user_watches:
# Smth about memory consumption
max_map_count:
# How soon to write to swap instead of RAM
swappiness:
# smth about TCP/IP connections
net_core_somaxconn:
```

### Notifications

#### Sudo mail

If a user who is not listed in the policy tries to run a command via sudo, mail is send to (if variable is defined):

```yaml
sudo_notification_mail:
```

#### SSH login mail

if `ssh_login_notification_mail` is set, an email with relevant info is sent on every SSH login via exim  
sends to `inventory_hostname@sender_email_domain` if `sender_email_domain` is defined, else to whatever default is configured  

command:  

```bash
{
  echo "User: $PAM_USER"
  echo "Remote Host: $PAM_RHOST"
  echo "Service: $PAM_SERVICE"
  echo "TTY: $PAM_TTY"
  echo "Date: $(date)"
  echo "Host: $(hostname -f)"
  echo "Server: $(uname -a)"
} | mail -s "$PAM_SERVICE login on $(hostname -f) for account $PAM_USER" \
{{ '-r ' + inventory_hostname + '@' + sender_email_domain if sender_email_domain is defined }} \
{{ ssh_login_notification_mail }}
```

#### SSH login webhooks

if `ssh_login_notification_webhooks` is set, the server tries to `curl` the given url on every SSH login  
Be careful, as you can block login with a wrong URL! Timeouts via `-m 10` are recommended  

remove old webhooks by adding `state: absent` (only `name:` required, see examples)  

```yml
ssh_login_notification_webhooks:
  - name: ""
    url: ""
    options: ""
    state: absent | present 
```

## License

MIT

## Author Information

FW-OSS, 2025
