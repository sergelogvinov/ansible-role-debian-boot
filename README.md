# Ansible role debian-boot

If your server does not have DHCP, PXE, or IPMI in your network, it can be challenging to automate the deployment of Debian operating systems.
In such cases, you may want to consider using an Ansible role to simplify the process.
This role downloads the Talos kernel and launch the OS by command `kexec -l <kernel-image> --initrd=<initrd-image> --append=<kernel-command-line-args>`.
It also sets the networks configuration in kernel parameters, witch helps you to boot OS with static IP.

Unfortunately, not all servers support booting a kernel through kexec.
Some servers may have hardware or firmware limitations that prevent the use of kexec, or the operating system may not have the necessary drivers or support for kexec.
In such cases, you can set the boot menu to boot the Debian Installer on the next boot and then install the debian.

## Install

```shell
ansible-galaxy role install git+https://github.com/sergelogvinov/ansible-role-debian-boot.git,main
```

## Options

Useful variables

1. ```debian_version: bookworm``` - version on distributive
2. ```debian_grub: true``` - add boot menu to the grub (boot menu)
3. ```debian_kexec: true``` - download and run talos
4. ```debian_interface: eth0``` - network interface
5. ```debian_rebuild_initrd: true``` - rebuild initrd image and add network configuration to it
6. ```debian_preseed: proxmox.cfg``` - preseed file name
    * proxmox.cfg - preseed file for proxmox with software raid1
    * proxmox-lvm.cfg - preseed file for proxmox with one disk

## Prepare and launch Debian Installer

```yaml
# debian.yml

- hosts: all
  vars:
    debian_grub: true
    debian_kexec: true
    debian_rebuild_initrd: true

    # If the server has bond interface
    debian_interface: bond0

    # debian_preseed: proxmox.cfg
    # debian_console: nomodeset console=tty0 console=ttyS0,115200n8
    # debian_cmdline_addon: "iommu=pt"
  roles:
    - ansible-role-debian-boot
```

```yaml
# debian.ini

[all]
debian   ansible_host=1.2.3.4 ansible_ssh_user=root # ansible_port=112233
```

Deploy debian to the server

```shell
ansible-playbook -Dv -i debian.ini debian.yml
```

## Disk partitioning

### proxmox.cfg (RAID1)

Partitions:
  * 1,2 - BIOS/EFI
  * 3 - Softraid + ext4 /boot
  * 4 - swap
  * 5 - Softraid + lvm

```shell
/dev/md0               943M  197M  682M  23% /boot
/dev/sdb2              512M  164K  512M   1% /boot/efi
/dev/mapper/data-root   20G  3.0G   17G  16% /
/dev/mapper/data-vz    3.8G   60M  3.7G   2% /var/lib/vz
```

### proxmox-lvm.cfg

Partitions:
  * 1,2 - BIOS/EFI
  * 3 - ext4 /boot
  * 4 - swap
  * 5 - lvm

```shell
/dev/sdb3              943M  197M  682M  23% /boot
/dev/sdb2              512M  164K  512M   1% /boot/efi
/dev/mapper/data-root   20G  3.0G   17G  16% /
/dev/mapper/data-vz    3.8G   60M  3.7G   2% /var/lib/vz
```

# Resources

- https://www.debian.org/releases/bookworm/example-preseed.txt
- https://preseed.debian.net/debian-preseed/bullseye/amd64-main-full.txt
- https://www.debian.org/releases/buster/amd64/apbs02.en.html
