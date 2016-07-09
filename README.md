# Ansible "Proof of Concepts" project

An ansible project that solves and documents some of the issues I've faced while using ansible, vagrant local vms, and remote servers together.

Keeping these examples in a simple, isolated project lets me refer to known working examples, in case I ever have issues while working in more complex projects.

## Inventory

See hosts/README.md for notes specific to inventory files.

## Playbooks

### vagrant.yml

This is a near-empty playbook that vagrant runs by default on `vagrant up`. We don't really want a playbook to be run automatically (we want to choose from one of the below playbooks), however, using the vagrant ansible provisioner requires that you choose a playbook, so we just give it this dummy playbook to keep it happy. You may ask "then why use the ansible provisioner at all?" - because we want it to auto-generate an inventory file for us.