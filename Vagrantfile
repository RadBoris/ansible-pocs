# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # disable vagrant-vbguest plugin
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.no_install = true
  end

  config.vm.box = "ubuntu/trusty64"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # disables the default synced folder - not used in this project
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  def fail_with_message(msg)
    fail Vagrant::Errors::VagrantError.new, msg
  end

  if Vagrant.has_plugin? 'vagrant-hostmanager'
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.hostmanager.manage_guest = false
  else
    fail_with_message "vagrant-hostmanager missing, please install the plugin with this command:\nvagrant plugin install vagrant-hostmanager"
  end

  config.vm.define "alpha" do |alpha|
    # don't use an ip that ends in '.1' - apache won't serve the default site through this ip.
    alpha.vm.network :private_network, ip: '192.168.1.2'
    alpha.hostmanager.aliases = ['alpha.local', 'alpha.dev']
  end

  config.vm.define "beta" do |beta|
    beta.vm.network :private_network, ip: '192.168.1.3'
    beta.hostmanager.aliases = ['beta.local', 'beta.dev']
  end

  config.vm.provision "ansible" do |ansible|

    ansible.playbook = "playbooks/vagrant.yml"
    ansible.groups = {
      "development" => ['alpha', 'beta'],
      "primary" => ['alpha'],
      "secondary" => ['beta']
    }
    
  end

end
