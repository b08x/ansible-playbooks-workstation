#!/usr/bin/env ruby
# frozen_string_literal: true

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

# Configuration variables
ALMALINUX_HOSTNAME = 'almalinux-test.syncopated.dev'.freeze
ARCH_HOSTNAME = 'arch-test.syncopated.dev'.freeze

Vagrant.configure('2') do |config|
  # Global SSH configuration
  config.ssh.insert_key = false
  config.ssh.forward_agent = true

  # Rocky Linux 9 VM for testing
  config.vm.define 'almalinux', primary: true do |almalinux|
    almalinux.vm.box = 'almalinux/10'
    almalinux.vm.hostname = ALMALINUX_HOSTNAME

    almalinux.vm.network :private_network,
                     ip: '192.168.122.10',
                     libvirt__network_name: 'default'

    almalinux.vm.provider :libvirt do |libvirt|
      libvirt.memory = 4096
      libvirt.uri = "qemu:///system"
      libvirt.cpus = 4
      libvirt.nested = true
      libvirt.disk_bus = 'virtio'
      libvirt.cpu_mode = 'host-passthrough'
      libvirt.nic_model_type = 'virtio'
      libvirt.disk_driver cache: 'writeback'
    end

    # Sync the entire project for testing
    almalinux.vm.synced_folder '.', '/vagrant',
                               type: 'rsync',
                               rsync__exclude: ['.git/', '*.swp', '.venv/', '.vagrant/']

    # Copy SSH key for testing
    almalinux.vm.provision 'file',
                           source: '~/.ssh/id_ed25519.pub',
                           destination: '/tmp/id_ed25519.pub'

    # Bootstrap system for testing
    almalinux.vm.provision 'shell', inline: <<-SHELL
      # Update system
      dnf update -y

      # Install required packages
      dnf install -y epel-release
      dnf install -y python3 python3-pip ansible-core git

      # Setup SSH key
      mkdir -p /home/vagrant/.ssh
      cat /tmp/id_ed25519.pub >> /home/vagrant/.ssh/authorized_keys
      chmod 600 /home/vagrant/.ssh/authorized_keys
      chown vagrant:vagrant /home/vagrant/.ssh/authorized_keys

      # Create test user that matches inventory
      useradd -m -s /bin/bash testuser || true
      echo "testuser ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/testuser
    SHELL

    # Run Ansible playbook for testing
    almalinux.vm.provision 'ansible' do |ansible|
      ansible.playbook = 'site.yml'
      ansible.inventory_path = 'inventory/hosts.ini'
      ansible.limit = 'almalinux-test.syncopated.dev'
      ansible.extra_vars = {
        ansible_python_interpreter: '/usr/bin/python3',
        ansible_user: 'vagrant',
        # Override variables for testing
        use_containers: 'false',
        use_kvm: 'false'
      }
      # ansible.tags = ENV['ANSIBLE_TAGS'] || 'base,ssh,shell'
      ansible.verbose = ENV['ANSIBLE_VERBOSE'] || false
      ansible.raw_arguments = ['--check'] if ENV['ANSIBLE_CHECK']
    end
  end

  # Optional Arch Linux VM for comparison testing
  config.vm.define 'arch', autostart: false do |arch|
    arch.vm.box = 'archlinux/archlinux'
    arch.vm.hostname = ARCH_HOSTNAME

    arch.vm.network :private_network,
                    ip: '192.168.122.11',
                    libvirt__network_name: 'default'

    arch.vm.provider :libvirt do |libvirt|
      libvirt.uri = 'qemu:///system'
      libvirt.memory = 2048
      libvirt.cpus = 2
      libvirt.nested = true
      libvirt.disk_bus = 'virtio'
      libvirt.cpu_mode = 'host-passthrough'
      libvirt.nic_model_type = 'virtio'
      libvirt.disk_driver cache: 'writeback'
    end

    arch.vm.synced_folder '.', '/vagrant',
                          type: 'rsync',
                          rsync__exclude: ['.git/', '*.swp', '.venv/', '.vagrant/']

    arch.vm.provision 'file',
                      source: '~/.ssh/id_ed25519.pub',
                      destination: '/tmp/id_ed25519.pub'

    arch.vm.provision 'shell', inline: <<-SHELL
      # Update system
      pacman -Syu --noconfirm

      # Install required packages
      pacman -Sy --noconfirm python python-pip ansible git

      # Setup SSH key
      mkdir -p /home/vagrant/.ssh
      cat /tmp/id_ed25519.pub >> /home/vagrant/.ssh/authorized_keys
      chmod 600 /home/vagrant/.ssh/authorized_keys
      chown vagrant:vagrant /home/vagrant/.ssh/authorized_keys
    SHELL

    # Run Ansible playbook
    arch.vm.provision 'ansible' do |ansible|
      ansible.playbook = 'site.yml'
      ansible.inventory_path = 'inventory/hosts.ini'
      ansible.limit = 'arch-test.syncopated.dev'
      ansible.extra_vars = {
        ansible_python_interpreter: '/usr/bin/python3',
        ansible_user: 'vagrant',
        use_docker: 'false',
        use_libvirt: 'false',
        window_manager: 'i3',
        rvm_install: false
      }
      ansible.tags = ENV['ANSIBLE_TAGS'] || 'base,ssh,shell'
      ansible.verbose = ENV['ANSIBLE_VERBOSE'] || false
      ansible.raw_arguments = ['--check'] if ENV['ANSIBLE_CHECK']
    end
  end
end

# Usage Examples:
#
# Basic testing:
#   vagrant up almalinux
#
# Test with specific tags:
#   ANSIBLE_TAGS="base,network" vagrant up almalinux
#
# Run in check mode (dry-run):
#   ANSIBLE_CHECK=true vagrant up almalinux
#
# Enable verbose output:
#   ANSIBLE_VERBOSE=true vagrant up almalinux
#
# Test both distributions:
#   vagrant up almalinux arch
#
# Re-run provisioning after changes:
#   vagrant provision almalinux
