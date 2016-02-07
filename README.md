# Ansible Cheatsheet
Cliff notes bro, I'm not a genius.

- [Definitions](#definitions) 
  - [The Basics](#the-basics)
  - [More Features](#more-features)
- [Example Layouts](#example-layouts)
  - [Simple](#simple)
  - [Advanced](#advanced)
  - [Host](#host)
  - [Playbook](#playbook)
- [Run Playbooks](#run-playbooks)
  - [Reboot One Server](#reboot-one-server)
  - [Ad-Hoc Commands](#ad-hoc-commands)
  - [Paralell Running](#paralell-running)
- [Security](#security)
  - [Encrypt a Password](#encrypt-a-password)
  - [Encrypt a Vault](#encrypt-a-vault)

# Definitions

### The Basics
| Format | Type | Description |
|---|---|---|
| ini | Inventory | file to describe a host |
| yml | Playbooks | The main configurations for each section |
| yml | Plays | A single item inside a playbook, eg: install nginx. |
| j2 | Templates | Templates with optional variable configurations in Jinja2 format |

### More Features
| Format | Type | Description |
|---|---|---|
| yml | Tags | Can add to almost anything, add to your Plays to SSH only certain items. |
| yml | Roles | Apply any Rules to different Playbooks |
| yml | Handlers | A separate area that accepts the "notify" command. |
| yml | Notify | Add to a Play and it will trigger whatever the Handler does. |

# Example Layouts

### Simple
```
/roles/
  /apache2/  <-- Folder format for all your roles
    /tasks/
      main.yml
vars.yml <-- optional
hosts
playbook.yml
```

### Advanced
```
/handlers/
  services.yml <-- Reboot Apache, SSH, etc
  .. Can have many
/roles/
  /apache2/
    /files/
      anything
    /tasks/
      main.yml (The main guy)
    /templates/
      vhost.j2
/vars/ (Or you can have one single vars file)
  apache.yml
hosts <-- INI format
playbook.yml <-- The main runner!
```

### Host
This would be called `host` in the root folder:
```
[websites]
dallas   site-a.com
boston   site-b.com:3333

[targets]
localhost       ansible_connection=local
site-a.com      ansible-connection=ssh
site-a.com      ansible_connection=ssh
```

### Playbook
A playbook runs roles, you might have something like this:
```
---
- name: Provision Webserver
  hosts: all
  user: root   # Deploying as root will require --ask-sudo-pass
  sudo: true
  gather_facts: yes  # http://docs.ansible.com/ansible/playbooks_variables.html
  vars_files:
    - vars/apache.yml
  vars:  # Dynamically set variables too
    - { is_vagrant: false }

  # Where all the rules run, you can optionally do this without the tags like:
  # roles:
  # - role: core
  # - role: host
  # - role: apache2
  roles:  
    - { role: core, tags: ['core'] }
    - { role: host, tags: ['host'] }
    - { role: apache2, tags: ['apache2', 'apache'] }

  # If you are using handlers to notify events
  handlers:
    - include: handlers/services.yml
```

# Run Playbooks
You can use `-vvv` and `-vvvv` for verbose output. In these cases pretend we have a `hosts` file with `production` group with two servers: `dallas` and `boston`.
  
```
  ansible-playbook -i dallas site.yml
  ansible-playbook -i dallas playbook.yml --ask-sudo-pass
```

```
  ansible-playbook -i dallas playbook.yml
  ansible-playbook -i -vvv dallas playbook.yml --tags anything_you_tagged
  ansible-playbook playbook.yml --skip-tags "anything_you_tagged"
```
  
### Reboot One Server
You could reboot only a boston server from the hosts file with:

```
  ansible-playbook -i production webservers.yml --limit boston
```

### Ad-Hoc Commands

```
  ansible boston -i production -m ping
  ansible boston -i production -m command -a '/sbin/reboot'
```
### Paralell Running

  ansible-playbook playbook.yml -f 10

#Security

### Encrypt a Password
Copy and paste the password you create from here:

```
   mkssh_userpass --method=SHA-512
```

### Encrypt a Vault
You can `pip install cryptography` for faster speeds.
To encrypt sensitive data run:

```
  ansible-vault encrypt keydata.yml
```

To decrypt it while running your playbook 

```
  ansible-playbook test.yml --ask-vault-pass
```  
