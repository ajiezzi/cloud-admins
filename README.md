## Ansible Role - cloud-admins

Ansible role which manages users, groups, and SSH keys across an Amazon Web Services (AWS) VPC.

## Role configuration

* For each user, the host group in the variable file `/vars/main.yml` must exactly match a host group from the ansible inventory file `/etc/ansible/hosts`.
* In the inventory file, all non-bastion instances MUST belong to the admin group

## Inventory File

The inventory file host group names `[groupA]` are used in classifying instances. Other than the bastion `[bastion]` group, each user must belong to one of the defined groups. 

Example:

	---
	[bastion]
	10.0.0.100

	# All non-bastion instances must belong to the admin group
	[admin]
	localhost ansible_connection=local
	10.0.0.101
	10.0.0.102
	10.0.0.103

	[groupA]
	10.0.0.101
	10.0.0.102

	[groupB]
	10.0.0.103

## Creating users

The `current_users` variable contains a list of current priviledged users across the VPC. Please note power users must belong to the admin host group. `host_group: admin` 

The following attributes are required for each user:

* name - The username of the user. Must be unique.
* host_group - The machine host group the user will belong to. Host groups must reference a group defined in the ansible inventory file.
* pub_key - The SSH public key of the specific user. 

Example:

    ---
    current_users:
      - name: jdoe
        host_group: groupA
        pub_key: "ssh-rsa AAAAA...."

## Removing users

The `removed_users` variable contains a list of users who should no longer have access to the VPC. Please note these users will be removed on the next ansible run. The format is similar to creating users, but the only required field is `name`.

Example:

    ---
    remove_users:
      - name: jdoe

## Example playbook to execute role

	---
	- hosts: all
      become: yes
  	  become_user: root
  	  connection: ssh
  	  gather_facts: no
  	  roles:
    	- cloud-admins

