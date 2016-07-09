# Inventory

Note: this file would normally be parsed as an inventory file! However, we have ignored .md files in ansible.cfg

## Why is the _development inventory file a symlink?

When you provision a vagrant machine with ansible, an inventory file is automatically generated at .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory. This is the inventory file that is used by `vagrant provision` and `vagrant ssh` to connect to the box.

When we come to add more environments like staging and production, it would be useful to have the details in this file replicated in our hosts directory. Then we can tell ansible.cfg to look in this one directory for connection information to all our hosts. In the interest of keeping things DRY, rather than duplicating this file, we add a symlink to it's original location. This way, it will stay updated if the details of the box ever change, or if we add multiple boxes.

By default, the vagrant_ansible_inventory doesn't add the hosts to any groups. However, you can add each box to as many groups you wish within the provisioner configuration in the Vagrantfile, like this:

```
ansible.groups = {
  'development' => ['default']
}
```

We can then reference the 'development' group in our playbooks.

## Why are some inventory file names prefixed with underscores?

This is a workaround for a problem. The contents of the hosts directory are processed in alphabetical order. This means that if you had inventory files 'a' and 'b', and in 'a' you tried to reference a group name that is defined in 'b', you would get an error like: `"Section [mysection:children] includes undefined group: mygroup"`

To work around this, we make sure all groups are defined first (hence adding the underscore to make them first alphabetically).

## What is the insecure_ssh inventory file?

(The insecure_ssh file is based on a solution found here: http://stackoverflow.com/a/35564773/3293805)

This file adds a nested inventory group that allows us to disable strict host key checking for a group of servers. Strict host key checking is how SSH determines whether the machine you're connecting to is the machine you really think it is, by checking its 'fingerprint'.

But why would we want to disable this?

Disabling string host key checking is particularly useful for the development group, the server for which (in this project) is built on vagrant. If you do a `vagrant destroy` followed by a `vagrant up`, you will have a brand new machine with a brand new 'fingerprint', but with the same IP address of 127.0.0.1. Thus, the next time you run the ansible-playbook command, the connection will fail, and you'll see an error message like:

> WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED! IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!

In fact, nothing nasty is happening, we've just rebuilt our machine. By disabling strict host key checking, we allow ansible to continue to connect to the machine.

Disabling string host key checking can also be useful if you test your playbooks against a remote host, and you occasionally rebuild that host (e.g. on AWS or Digital Ocean). For this reason, this project also disables strict host key checking for the 'staging' group.

To disable strict host key checking for a group, add the group name under `[insecure_ssh:children]` in the 'insecure_ssh' file. This should not be done on production, or indeed any server you plan to deploy sensitive information to (which might include staging). Use at your own risk.

Note:
* Running `vagrant provision` after rebuilding your vagrant machine does not throw up the above warning, so disabling strict host key checking is only necessary if you're trying to run ansible directly, e.g. via `ansible-playbook`. Likewise, `vagrant ssh` does not throw up the warning.
* Disabling strict host key checking in ansible does not have an effect when SSH'ing into the machine using the ssh command - you will see the above warning. A workaround in this case is to remove the old 'fingerprint' from your ~/.ssh/known_hosts file. Look for the IP address of the machine, and remove that line from the file.
