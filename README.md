# Ansible "Proof of Concepts" project

An ansible project that solves and documents some of the issues I've faced while using ansible, vagrant local vms, and remote servers together.

Keeping these examples in a simple, isolated project lets me refer to known working examples, in case I ever have issues while working in more complex projects.

## Ansible version

New versions of Ansible are released regularly. The playbooks in this project are always tested against a specific version, which is specified in requirements.txt (e.g. `ansible==2.2.1.0`).

## Server OS version

These playbooks are tested against:

* Vagrant boxes running a specific box version as specified in 'Vagrantfile', e.g. `config.vm.box_version = "v20170202.1.0"`
* Digital Ocean machines running Ubuntu 14.04.5 x64

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

### rsync.yml

A playbook to sync a folder on one remote host, to a folder on another remote host.

Note: Add your own remote hosts to the inventory first (see hosts/_staging.example). Also run remote-admin-user.yml on both servers you wish to sync, before running this playbook. This adds the admin user, and your local public key, to the servers.

Currently tested on two remote hosts (not vagrant).

View the comments in the playbook for more info.

### ansible-pull-setup.yml

A playbook for setting up an [ansible-pull](http://docs.ansible.com/ansible/playbooks_intro.html#ansible-pull) scenario. 

`ansible-pull` is a command that inverts the usual ansible architecture from a 'push' mode to a 'pull' mode. Instead of having a control machine that SSH's into your remote server and runs the tasks, the remote server itself will pull an ansible project from a git repo, and run it in place on the server.

This can be particularly useful for cron jobs. While ansible has a [cron module](http://docs.ansible.com/ansible/cron_module.html) for scheduling cron jobs on a server, it is less obvious as to how one should write the script that performs the actual job. You might think that writing a .sh script is the best solution, but having already invested time into learning ansible, wouldn't it be great if you could define your cron jobs as ansible playbooks? `ansible-pull` allows you to do just that.

`ansible-pull` can pull from any public repo you specify (and probably a private repo too, with some extra config on the server). However, it makes sense to keep all the code for a project together, so this project assumes we'll be pulling the very same repo that stores all the rest of our tasks - i.e. the one you're looking at now.

#### Step 1: prepare the server and schedule the job

This is a two-part process. Firstly, you need a playbook that is going to prepare your server to run the `ansible-pull` command, and schedule a cron job to run that command. This involves installing git and ansible, and using the cron module to schedule the job. This is demonstrated in playbooks/ansible-pull-setup.yml.

The tricky part is choosing the correct options for the `ansible-pull` command. Here's the formula used in this project:

`ansible-pull --directory {{ workdir }} --url {{ repo_url }} --checkout {{ repo_branch }} --full --force -i '127.0.0.1,' playbooks/ansible-pull-action.yml`

The important flags here are:
* `--full` - checks out the full repo. This seems to only be necessary if checking out a branch other than master.
* `-i '127.0.0.1,'` - the inventory to use. This can either be a directory, file, or a comma-separated list of hostnames. Ideally we would not supply this flag at all, and instead default to the inventory specified in ansible.cfg. Unfortunately, doing so results in an error about a missing file, due to a symlink with a missing destination (see hosts/README.md for more info on why we have symlink here). Thus, we opt for the simplest option of specifying the host directly. Cron jobs usually take care of some maintenance on the host they run on, so the host we specify is '127.0.0.1,'. We could still delegate some jobs to other hosts with `delegate_to` if we wanted to. The comma is important here, as otherwise ansible-pull looks for a file of this name, rather than treating it as a list of hostnames.
* --force - forces the playbook to run even if the repo couldn't be pulled this time (using the last sucessfully pulled version).

#### Step 2: write the job

The second part is to define the job itself as an ansible playbook. Remember that this playbook will be running from the remote host itself, and thus any tasks that should operate on the host itself should not use the external IP, but the internal IP. Hence, we set `hosts: 127.0.0.1`. In all other respects, writing the tasks is exactly the same as when running ansible from the control machine. This is demonstrated in playbooks/ansible-pull-action.yml.

## TODO

## DONE
* ~~Add a POC for ansible-pull~~
