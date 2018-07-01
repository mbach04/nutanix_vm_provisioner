Nutanix VM Provisioner
=========

A basic Ansible role to provision virtual machines on a Nutanix AHV using APIv3.

Required Variables
------------
```
api_version: "3.0"          # Nutanix is moving away from v2 api, it's best to use v3 if available
cluster_url: 172.16.1.100   # IP address where you would normally log into PRISM
prism_user: admin           # An account with permissions to provision on the cluster
prism_password: secret      # The password to your account, Note, you should not store this in the clear, use Ansible vault

cluster_name: "Your Cluster"      # Name of the cluster to provision against
subnet_name: "VMNet"              # Name of the Subnet (vlan) to add VMs to
image_name: "RHEL_Server_7.5"     # Name of the disk image or ISO to use

# A list of dicts that define the VMs you want created
# Note, you can break this into several separate lists and override vm_defs when calling the role to loop across
# several sets of VMs.
vm_defs:
  - {vm_name: my-vm-01, vm_ip: '172.16.1.111', vm_ram: 8192, vm_num_cpu_per_socket: 1, vm_num_sockets: 1, vm_disk_list: [disk_size_mib: 152588]}
```


Cloud-init
----------
If you do not wish to use guest customization (cloud-init in this case), then remove the `guest_customization` section from the `templates/vm-body.yml.j2` file, or variablize it out and submit your change to this repo. :-)

If you wish to use the role with a cloud-init script and set a user password, you can (but it's not the best security practice).

Use the following command on a RHEL host to generate a SHA-512 hashed password to be cloud_init_root_pass used with the `kvm` RHEL image.

`python -c 'import crypt,getpass; print crypt.crypt(getpass.getpass())'`

Set the resulting string equal to `cloud_init_root_pass` in `group_vars/*/all.yaml`.

Example Playbook
----------------
```
---
- name: Provision some VMs
  hosts: localhost
  gather_facts: false
  vars:
    my_vms:
      - {vm_name: server1, vm_ip: '172.29.171.100', vm_ram: 4096, vm_num_cpu_per_socket: 1, vm_num_sockets: 4, vm_disk_list: [disk_size_mib: 76294, disk_size_mib: 152588]}
      - {vm_name: server1, vm_ip: '172.29.171.100', vm_ram: 4096, vm_num_cpu_per_socket: 1, vm_num_sockets: 4, vm_disk_list: [disk_size_mib: 76294, disk_size_mib: 152588]}
  tasks:
    - name: Provision VMs on Nutanix
      include_role:
        name: nutanix_provisioner
      vars:
        vm_defs: "{{ my_vms }}"
```



License
-------

Licensed under the MIT License. See the LICENSE file for details.


Author Information
------------------

Created by Matthew Bach and Timothy Ling, from Red Hat
