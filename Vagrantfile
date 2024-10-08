# -*- mode: ruby -*-
# vi: set ft=ruby :

# Load override settings
require 'yaml'
settings_path = '.settings.yml'
settings = {}

if File.exist?(settings_path)
  settings = YAML.load_file(settings_path)
end

# Default Vagrant settings
VAGRANT_BOX = settings['VAGRANT_BOX'] || 'debian/bookworm64'
VAGRANT_CPU = settings['VAGRANT_CPU'] || 2
VAGRANT_MEM = settings['VAGRANT_MEM'] || 2048
SSH_FORWARD = settings['SSH_FORWARD'] || false

# Default Slurm settings
SLURM_NODES  = settings['SLURM_NODES']  || 4
JOIN_TIMEOUT = settings['JOIN_TIMEOUT'] || 120

# Node settings overrides
if File.exist?('nodes.rb')
  require_relative 'nodes.rb'
else
  # Using all defaults
  NODES = {}
end

HOSTS = Array(1..SLURM_NODES)
Vagrant.configure(2) do |vm_config|
  HOSTS.each do |count|
    vm_config.vm.define "node#{count}" do |config|
      config.vm.hostname = "node#{count}"
      config.vm.box = NODES.dig("node#{count}", 'BOX') || VAGRANT_BOX
      config.ssh.forward_agent =
        NODES.dig("node#{count}", 'SSH') || SSH_FORWARD

      # Libvirt
      config.vm.provider :libvirt do |virt|
        virt.memory = NODES.dig("node#{count}", 'MEM') || VAGRANT_MEM
        virt.cpus = NODES.dig("node#{count}", 'CPU') || VAGRANT_CPU
      end

      # Install and Setup Slurm
      config.vm.provision "shell", inline: <<-SHELL
        export JOIN_TIMEOUT=#{JOIN_TIMEOUT}
        /bin/bash /vagrant/provision.sh
      SHELL
    end
  end
end
