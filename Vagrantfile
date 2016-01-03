# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrant LEMP Ansible
# A PHP7 development platform in a box.
# See the readme file (README.md) for more information.

# If vagrant-trigger isn't instaled then exit
if !Vagrant.has_plugin?("vagrant-triggers")
  puts "'vagrant-triggers' plugin is required"
  puts "This can be installed by running:"
  puts
  puts " vagrant plugin install vagrant-triggers"
  puts
  exit
end

# Find the current vagrant directory.
vagrant_dir = File.expand_path(File.dirname(__FILE__))
provision_hosts_file = vagrant_dir + '/host.ini'

# Include config from settings.yml
require 'yaml'
vconfig = YAML::load_file(vagrant_dir + "/settings.yml")

# Configuration
boxipaddress = vconfig['boxipaddress']
boxname = vconfig['boxname']

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Configure virtual machine options.
  config.vm.box = "ubuntu/trusty64"
  config.vm.hostname = boxname

  config.vm.network :private_network, ip: boxipaddress

  # Set *Vagrant* VM name
  config.vm.define boxname do |boxname|
  end

  # Configure virtual machine setup.
  config.vm.provider :virtualbox do |v|
    v.customize ["modifyvm", :id, "--memory", vconfig["memory"]]
    v.customize ["modifyvm", :id, "--cpus", vconfig["cpus"]]
    v.customize ["modifyvm", :id, "--ioapic", "on"]
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  end

  # Set up NFS drive.
  nfs_setting = RUBY_PLATFORM =~ /darwin/ || RUBY_PLATFORM =~ /linux/

  # SSH Set up.
  config.ssh.forward_agent = true

  # Run an Ansible playbook on setting the box up
  if !File.exist?(provision_hosts_file)
    config.trigger.before :up, :stdout => true, :force => true do
      run 'ansible-playbook -i ' + boxipaddress + ', --ask-sudo-pass ' + vagrant_dir + '/playbooks/local_up.yml --extra-vars "local_ip_address=' + boxipaddress + '"'
    end
  end

  # Run the halt/destroy playbook upon halting or destroying the box
  if File.exist?(provision_hosts_file)
    config.trigger.before [:halt, :destroy], :stdout => true, :force => true do
      run 'ansible-playbook -i ' + boxipaddress + ', --ask-sudo-pass ' + vagrant_dir + '/playbooks/local_halt_destroy.yml'
    end
  end

  # Provision vagrant box with Ansible.
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = vagrant_dir + "/playbooks/setup.yml"
    ansible.host_key_checking = false
    ansible.extra_vars = {user:"vagrant"}
    if vconfig['ansible_verbosity'] != ''
      ansible.verbose = vconfig['ansible_verbosity']
    end
  end

  # Create synced folders for each virtual host.
  vconfig['vhosts'].each do |vhost|
    site_alias = vhost['alias']
    folder_path = vhost['path']
    config.vm.synced_folder folder_path,
      vconfig['sites_dir'] + "/" + site_alias,
      create: true,
      type: "nfs",
      :nfs => nfs_setting
  end

  # Create all virtual hosts
  config.trigger.after [:up, :reload], :stdout => true, :force => true do
    run 'ansible-playbook -i ' + boxipaddress + ', -K --user=vagrant --private-key=' + vagrant_dir + '/.vagrant/machines/' + boxname + '/virtualbox/private_key ' + vagrant_dir + '/playbooks/virtual_hosts.yml --extra-vars "local_ip_address=' + boxipaddress + '"'
  end

end
