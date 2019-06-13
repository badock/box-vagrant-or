#-*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

# IP Network to use:
NETWORK_PREFIX = ENV["OARVAGRANT_NETWORK_PREFIX"] || "192.168.37"

# NODES_COUNT set the number of nodes of your cluster.
# Change the value there or set the OARVAGRANT_NODES_COUNT environment variable in the calling shell
# ***Warning***: vagrant is very greedy on disk, consider using the trick below if
# running more than a few nodes.
NODES_COUNT = (ENV["OARVAGRANT_NODES_COUNT"] || 1).to_i

VM_MEMORY = (ENV["OARVAGRANT_VM_MEMORY"] || 512).to_i
VM_CPU = (ENV["OARVAGRANT_VM_CPU"] || 1).to_i
OAR_FTP_HOST = ENV["OAR_FTP_HOST"] || "oar-ftp.imag.fr"
OAR_FTP_DISTRIB = ENV["OAR_FTP_DISTRIB"]
DEBIAN_EXTRA_DISTRIB = ENV["DEBIAN_EXTRA_DISTRIB"]

## Uncomment the following lines to use VirtualBox Linked Clone VMS trick
## and optimze the storage size of the VMs
#require "ffi"
#if FFI::Platform::IS_LINUX
#  CUSTOM_PATH = File.join(File.dirname(__FILE__), "..", "misc", "vagrant-use-linked-clones")
#  ENV["PATH"] = "#{File.absolute_path(CUSTOM_PATH)}:#{ENV["PATH"]}"
#end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "oar-team/debian9"
  #config.vm.box_url = "http://oar-ftp.imag.fr/vagrant/debian8.box"
  config.ssh.insert_key = false
  config.disksize.size = '20GB'
  config.vm.define "server" do |m|
    m.vm.hostname = "server"
    #m.vm.disksize.size = '40GB'
    m.vm.network :private_network, ip: "#{NETWORK_PREFIX}.10"
    m.ssh.forward_agent = true
    m.vm.provision :shell, path: "provision.sh", args: "server #{NETWORK_PREFIX} #{NODES_COUNT} #{OAR_FTP_HOST} \"#{OAR_FTP_DISTRIB}\" \"#{DEBIAN_EXTRA_DISTRIB}\"", privileged: true
  end
  config.vm.define "frontend", primary: true do |m|
    m.vm.hostname = "frontend"
    m.vm.network :private_network, ip: "#{NETWORK_PREFIX}.11"
    m.ssh.forward_agent = true
    m.vm.provision :shell, path: "provision.sh", args: "frontend #{NETWORK_PREFIX} #{NODES_COUNT} #{OAR_FTP_HOST} \"#{OAR_FTP_DISTRIB}\" \"#{DEBIAN_EXTRA_DISTRIB}\"", privileged: true
  end
  (1..NODES_COUNT).each do |i|
    config.vm.define "node-#{i}" do |m|
      m.vm.hostname = "node-#{i}"
      m.vm.network :private_network, ip: "#{NETWORK_PREFIX}.#{i+100}"
      m.ssh.forward_agent = true
      m.vm.provision :shell, path: "provision.sh", args: "nodes #{NETWORK_PREFIX} #{NODES_COUNT} #{OAR_FTP_HOST} \"#{OAR_FTP_DISTRIB}\" \"#{DEBIAN_EXTRA_DISTRIB}\"", privileged: true
    end
  end

  config.vm.provider "virtualbox" do |v|
    v.memory = VM_MEMORY
    v.cpus = VM_CPU
    v.linked_clone = true if Gem::Version.new(Vagrant::VERSION) >= Gem::Version.new('1.8.0')
  end

  if Vagrant.has_plugin?("vagrant-proxyconf")
    # install polipo
    # vagrant plugin install vagrant-proxyconf
    config.proxy.http     = "http://#{NETWORK_PREFIX}.1:3128/"
    config.proxy.https    = "http://#{NETWORK_PREFIX}.1:3128/"
    config.proxy.no_proxy = "localhost,127.0.0.1"
  end

  if Vagrant.has_plugin?("vagrant-cachier")
    # vagrant plugin install vagrant-cachier
    config.cache.scope = :box
  end

end
