# archfiend

Ansible playbook to install my personal computer from scratch, including the partitioning and bootloader config. (uses full disk)
Forked from https://github.com/33Fraise33/desktop-ansible, which in turn is heavily based on: https://github.com/jsf9k/ansible-arch-install

## iso-install

Boot the host you want to manage with an arch iso. run the following commands:

1. `passwd` ("root" is used for the arch iso password, can be changed in the inventory files)
2. `systemctl start sshd`
3. `ip a` (to get the host ip address, change this in the inventory file)
4. copy contents of your ssh public key into ~/.ssh/authorized_keys **optional: In my environment, I use ssh keys from the get-go because of my 1Password configuration**

Change the variables you want under group_vars or settings.yaml. (depending if the variable is needed both in the postinstall playbook and the iso_install playbook)

In settings.yaml:
- `install_drive`: drive that will be wiped and used for installing arch. 2 partitions will be made, one for boot and one root partition.

In group_vars:
- `user`:
  - `name`: user to create that is not root
  - `password`: encrypted user password, can be clear text as well but this is not advised
  - `pub_key_location`: public key to add to host so we can run the common playbook after that
- `encryption`: set encryption with luks (luks1 as luks2 is not supported with grub)
  - `enabled`: true or false
  - `password`: encryption password, a random key will be generated for system load so you only have to enter your encryption password once
- `btrfs`: have the option to use btrfs as filesystem (includes subvolumes)
  - `enabled`: true or false
  - `subvols`: list containing subvol name and map location
    - `sub`: i.e. @log
    - `path`: i.e. var/log
- `manual_partitions`: override default wipe behaviour (both options necessary, you may overwrite an existing boot partition if wanted)
  - `boot`: set custom boot partition location (i.e. nvme0n1p1)
  - `root`: set custom root partition locatoin (i.e. nvme0n1p3)
- `hostname`: sets hostname for networking
- `timezone`: timezone to set on the host

Run the playbook:
`ansible-playbook -i settings.yaml iso-installation.yaml`

*remarks*:
- currently locale is hard set
- currently pacman reflector is using US as location
- grub is set to efi

## common
this relies on ansible-aur (https://github.com/kewlfft/ansible-aur) and ansible-role-mkinitcpio (https://github.com/aisbergg/ansible-role-mkinitcpio)

ansible-aur is included in the arch install ansible package, and does not need any additional configuration.
ansible-role-mkinitcpio should be installed via ansible_galaxy.  e.g. ansible-galaxy install -r requirements.yaml

Items set in `iso-install` can be overridden for hosts that are maybe not provisioned with Ansible.

### Running on Arch ARM
When running on arch arm use the following method to run
1) change the setting in `/etc/ssh/sshd_config` to `PermitRootLogin yes`
(this will be undone in our playbook during the user setup)
2) restart sshd `systemctl restart sshd`
3) run: `ansible-playbook -i production.yaml common.yaml -e "ansible_user=alarm" --ask-pass --ask-become-pass --become-method su --tags arm_setup`

## WSL Prerequisites

Run before ansible:
- https://github.com/yuk7/ArchWSL/wiki/How-to-Setup
- pacman -S openssh
- systemctl start sshd
