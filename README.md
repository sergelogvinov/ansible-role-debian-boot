# Ansible role debian-boot

If your server does not have DHCP, PXE, or IPMI in your network, it can be challenging to automate the deployment of Debian operating systems.
In such cases, you may want to consider using an Ansible role to simplify the process.
This role downloads the Talos kernel and launch the OS by command `kexec -l <kernel-image> --initrd=<initrd-image> --append=<kernel-command-line-args>`.
It also sets the networks configuration in kernel parameters, witch helps you to boot OS with static IP.

Unfortunately, not all servers support booting a kernel through kexec.
Some servers may have hardware or firmware limitations that prevent the use of kexec, or the operating system may not have the necessary drivers or support for kexec.
In such cases, you can set the boot menu to boot the Talos kernel on the next boot and then reboot the server.

## Install

```shell
ansible-galaxy role install git+https://github.com/sergelogvinov/ansible-role-debian-boot.git,main
```

## Options

Useful variables

1. ```debian_version: bookworm``` - version on distributive
2. ```debian_grub: true``` - add boot menu to the grub (boot menu)
3. ```debian_kexec: true``` - dowwnload and run talos
4. ```debian_interface: eth0``` - network interface

## Launch Talos

```yaml
# talos.yml

- hosts: all
  vars:
    debian_grub: true
    debian_kexec: true

    # IPv6 network (will work from version 1.4.0)
    #
    # debian_cmdline_net: "ip=[{{ ansible_default_ipv6['address'] }}]::[{{ ansible_default_ipv6['gateway'] }}]:{{ ansible_default_ipv6['prefix'] }}::{{ debian_interface }}:off:[2001:4860:4860::8888]:[2606:4700::1111]:[2606:4700:f1::1]"
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
