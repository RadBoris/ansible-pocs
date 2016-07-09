# Ansible "Proof of Concepts" project

An ansible project that solves and documents some of the issues I've faced while using ansible, vagrant local vms, and remote servers together.

Keeping these examples in a simple, isolated project lets me refer to known working examples, in case I ever have issues while working in more complex projects.

## Inventory

See hosts/README.md for notes specific to inventory files.

## Playbooks

### vagrant.yml

This is a near-empty playbook that vagrant runs by default on `vagrant up`. We don't really want a playbook to be run automatically (we want to choose from one of the below playbooks), however, using the vagrant ansible provisioner requires that you choose a playbook, so we just give it this dummy playbook to keep it happy. You may ask "then why use the ansible provisioner at all?" - because we want it to auto-generate an inventory file for us.

### remote-admin-user.yml

A playbook to add an 'admin' user to your remote machine, so you don't have to use root.

Most vagrant boxes come with a default 'vagrant' user with passwordless sudo. On the other hand, most remote VPS's come with root access only. It's better to run things as a standard user, and only elevate to root when necessary.

In this playbook, the very first role is to add the admin user - this is done while logging in as root (set as `remote_user` at task level in the admin_user role, to override the playbook level setting described below). This only needs to be done on remote machines though, so the 'development group' is excluded from this role.

Any roles or tasks beyond this should use the standard user. This is set at playbook level as `remote_user`. This in turn is set via a group_var called `my_remote_user` - because the user will be different depending on the environment ('vagrant' on the development box, 'admin' elsewhere).