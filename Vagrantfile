# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vlad - Vagrant LAMP Ansible Drupal
# A Drupal development platform in a box, with everything you would need to develop Drupal websites.
# See the readme file (README.md) for more information.
# Contribute to this project at : https://bitbucket.org/philipnorton42/vlad

# Find the current vagrant directory.
vagrant_dir = File.expand_path(File.dirname(__FILE__))
vlad_hosts_file = vagrant_dir + '/vlad/host.ini'

# Include config from vlad/settings.yml
require 'yaml'
vconfig = YAML::load_file("vlad/settings.yml")

# Configuration
boxipaddress = vconfig['boxipaddress']
boxname = vconfig['boxname']

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Configure virtual machine options.
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"
  config.vm.box = "vlad"
  config.vm.hostname = boxname

  config.vm.network :private_network, ip: boxipaddress

  # Allow caching to be used (see the vagrant-cachier plugin)
  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :machine
    config.cache.synced_folder_opts = { type: :nfs }
    config.cache.auto_detect = false
    config.cache.enable :apt
    #config.cache.enable :gem
    #config.cache.enable :npm
  end

  # Configure virtual machine setup.
  config.vm.provider :virtualbox do |v|
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--memory", 1024]
  end

  # Set up NFS drive.
  nfs_setting = RUBY_PLATFORM =~ /darwin/ || RUBY_PLATFORM =~ /linux/

  # Setup synced folder for site files
  config.vm.synced_folder vconfig['host_synced_folder'],
    "/var/www/site/docroot", 
    id: "vagrant-root",
    :nfs => nfs_setting,
    create: true

  # Setup auxiliary synced folder
  config.vm.synced_folder "./aux",
    "/var/www/site/aux",
    id: "vagrant-root",
    :nfs => nfs_setting

  # SSH Set up.
  config.ssh.forward_agent = true

  # Set machine name.
  config.vm.define :vlad do |t|
  end

  # Rung an Ansible playbook on setting the box up
  if Vagrant.has_plugin?("vagrant-triggers")
    if !File.exist?(vlad_hosts_file)
      config.trigger.before :up, { :execute => 'ansible-playbook -i ' + boxipaddress + ', --ask-sudo-pass vlad/playbooks/local_up.yml --extra-vars "local_ip_address=' + boxipaddress + '"', :stdout => true }
    end
  end

  # Run the destroy playbook upon halting the box
  if Vagrant.has_plugin?("vagrant-triggers")
    if File.exist?(vlad_hosts_file)
      config.trigger.before :halt, { :execute => "ansible-playbook -i vlad/host.ini --ask-sudo-pass vlad/playbooks/local_destroy.yml", :stdout => true }
    end
  end

  # Add an Ansible playbooks that executes when the box is destroyed to clear things up.
  if Vagrant.has_plugin?("vagrant-triggers")
    if File.exist?(vlad_hosts_file)
      config.trigger.before :destroy, { :execute => "ansible-playbook -i vlad/host.ini --ask-sudo-pass vlad/playbooks/local_destroy.yml", :stdout => true }
    end
  end

  # Provision vagrant box with ansible.
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "vlad/playbooks/site.yml"
    ansible.host_key_checking = false
    ansible.extra_vars = {user:"vagrant"}
    # Optionally allow verbose output from ansible.
    # ansible.verbose = 'vvvv'
  end

end
