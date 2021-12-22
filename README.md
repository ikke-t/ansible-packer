# Ansible Packer Role

[![License: GPLv2](https://img.shields.io/badge/license-GPLv2-brightgreen.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)
[![License: GPLv3](https://img.shields.io/badge/license-GPLv3-brightgreen.svg)](https://www.gnu.org/licenses/gpl-3.0)

Simple Ansible setup to build RHEL images with Packer.

## Quick Usage Example

```
### Build latest RHEL 8 image on Qemu with all defaults

Create playbook e.g. build_rhel_qemu.yml:

```
---

- name: Build RHEL image with Packer
  hosts: all
  become: true
  tasks:

    - debug:
        var: cloud_user_password

    - name: build rhel template for qemu
      vars:
        packer_builder: qemu
        packer_target: rhel_8_5
        boot_password: "{{ cloud_user_password }}"
        root_password: "{{ cloud_user_password }}"
        partitioning: auto
        disable_ipv6: true
        # security_profile: cis_server_l1
        image_name: rhel_8_5_image
        root_ssh_key: ssh-ed25519 xxxyyyzzz admin@coollab
        output_directory: /VirtualMachines/templates
        disk_size: 4096
        iso:
          rhel_8_5:
            url: "file:///home/itengval/VirtualMachines/rhel-8.5-x86_64-dvd.iso"
            checksum: sha256:1f78e705cd1d8897a05afa060f77d81ed81ac141c2465d4763c0382aa96cadd0
      include_role:
        name: ansible-packer
      tags: packer
```

run playbook:

```
ansible-playbook -i localhost, -c local build_rhel_qemu.yml
```

### Build latest RHEL 8 image on VMware with all defaults

```
---
- name: Build RHEL template with Packer
  hosts: all
  become: true
  tasks:

    - name: build rhel template for vmware
      vars:
        vsphere_server: i-wish-this-was-openshift.mylab.com
        vcenter_datacenter: MYDC
        vcenter_folder: MYDC/Redhat
        vcenter_cluster: LAB
        vcenter_resource_pool:
        vcenter_datastore: SVC-LUN-1
        vsphere_network: vlan2102

        packer_builder: vmware
        packer_target: rhel_8_5
        boot_password: "{{ cloud_user_password }}"
        root_password: "{{ cloud_user_password }}"
        partitioning: auto
        disable_ipv6: true
        # security_profile: cis_server_l1
        image_name: rhel_8_5_image
        root_ssh_key: ssh-ed25519 xxxyyyzzz admin@coollab
        # disk_size: 4096
        iso:
          rhel_8_5:
            url: "file:///home/itengval/VirtualMachines/rhel-8.5-x86_64-dvd.iso"
            checksum: sha256:1f78e705cd1d8897a05afa060f77d81ed81ac141c2465d4763c0382aa96cadd0
      include_role:
        name: ansible-packer
```

run playbook:

```
ansible-playbook -i localhost, -c local build_rhel_vmware.yml
```

## Introduction

A simple [Ansible](https://www.ansible.com/) setup to allow quick
building of
[RHEL](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux).
(and other) images with [Packer](https://www.packer.io/) using either
[Qemu](https://www.packer.io/docs/builders/qemu) or
[VMware vSphere](https://www.packer.io/docs/builders/vsphere/vsphere-iso)
as builders.

Adding support for other builders and operating systems as well as
further customization and parametrization of images is made easy by the
suitable split of configuration files and templates. Review the files
under [vars/](vars/) and [templates/](templates/) to see how to
customize or add support for new builders and operating systems.

These templates leave _root_ SSH login permitted! In most environments
this should be changed after initial development and testing phase.

The following table shows currently supported user-provided variables.
The first two paramaters, _packer\_builder_ and _packer\_target_ are
mandatory, the rest are optional.

| Variable         |  Allowed Values  |  Default  |
|:-----------------|:----------------:|:---------:|
| packer_builder   |  qemu, vmware    |   Unset   |
| packer_target    |  See 1) below    |   Unset   |
| packer_output    |      Path        |   See 2)  |
| boot_password    |  Bootloader pw   |   Unset   |
| root_password    |  root password   |   foobar  |
| bios_uefi\_boot  |  true or false   |   See 3)  |
| partitioning     |  See 4) below    |   See 4)  |
| disable_ipv6     |  true or false   |   false   |
| security_profile |  See 5) below    |   Unset   |
| image_name       |  Name for image  |  OS name  |

1. Use [vars/content_iso.yml](vars/content_iso.yml) to add support
   for additional OS versions.
2. See [vars/builder_qemu.yml](vars/builder_qemu.yml) and
   [vars/builder_vmware.yml](vars/builder_vmware.yml) for default
   output values.
3. Create BIOS/UEFI bootable image, otherwise support only the
   platform mode used for building the image.
4. Adjust installer configuration files such as
   [templates/cfg-rhel_8.j2](templates/cfg-rhel_8.j2)
   to add support for different partitioning layouts.
5. The value is passed as-is to the installer
   [OpenSCAP](https://www.open-scap.org/) module.

Additionally, using `-e do_cleanup=true` will delete anything left
behind on the build host by earlier builds. Using `-e use_force=true`
adds the `-force` parameter to Packer command line.

NB. CentOS/RHEL 7 fails to boot if initramfs is updated during build.

## See Also

See also
[https://github.com/myllynen/rhel-image](https://github.com/myllynen/rhel-image).

## License

GPLv2+
