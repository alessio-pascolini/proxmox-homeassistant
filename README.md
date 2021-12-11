# Home Assistant deploy on proxmox
Automate the installation of Home Assistant with Ansible

Using Ansible to import the official image of Home Assistant on Proxmox.
Ansible creates a kvm virtual machine with proper configuration, then it imports the Home Assistant image file attaching it as scsi disk 0.
The import is skipped when a vm named as value defined in ‘ha_vm_name’ exists on proxmox node.
Root user is required to import the home assistant image through ssh with key-based authentication.

## Requirements

* python proxmoxer
* python requests
* ansible

## Configure ansible

Import ssh private key into ssh-agent and create the encrypted file `host_vars/pve/vault`
with the following command. Choose a password to encrypt the file and insert username
and password of pve:

```bash
$ ansible-vault create host_vars/pve/vault
```

`host_vars/pve/vault`
```yaml
---
vault_pve_api_user: "root@pam"
vault_pve_api_password: "password"
...
```

Edit `host_vars/pve/vars` according your configuration:

```yaml
---
ansible_host: "192.168.1.1"
ansible_ssh_user: root
pve_api_user: "{{ vault_pve_api_user }}"
pve_api_password: "{{ vault_pve_api_password }}"
pve_node: "pve"
vm_storage_id: "local-zfs"
ha_version: "6.6"
ha_vm_name: "homeassistant"
ha_image_url: "https://github.com/home-assistant/operating-system/releases/download/{{ ha_version }}/haos_ova-{{ ha_version }}.qcow2.xz"
ha_comp_image_name: "{{ ha_image_url | basename }}"
ha_image_name: "{{ ha_comp_image_name | splitext | first }}"
...
```

## Install home assistant

Run ansible to install Home Assistant, insert the password used before to encrypt the vault file:

```bash
$ ansible-playbook --ask-vault-pass install_home_assistant.yml
```
