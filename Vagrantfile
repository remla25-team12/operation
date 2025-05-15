# -*- mode: ruby -*-
# vi: set ft=ruby :

# Variables
NUM_WORKERS = 2

CTRL_CPU = 1
CTRL_MEMORY = 4096
CTRL_IP = "192.168.56.100"

WORKER_CPU = 2
WORKER_MEMORY = 6144
WORKER_IP_BASE = "192.168.56."
WORKER_IP_LIST = (1..NUM_WORKERS).map { |num| "#{WORKER_IP_BASE}#{100 + num}" }

GENERAL_PLAYBOOK_PATH = "provisioning/general.yaml"
CTRL_PLAYBOOK_PATH = "provisioning/ctrl.yaml"
NODE_PLAYBOOK_PATH = "provisioning/node.yaml"


# VM setup function
def vm_setup(vm, name:, ip:, memory:, cpus:, playbook:)
  # Create VM (Step 1 & 2)
  vm.vm.box = "bento/ubuntu-24.04"
  vm.vm.hostname = name
  vm.vm.network "private_network", ip: ip # Add second NIC for host-only networking
  vm.vm.provider "virtualbox" do |vb|
    vb.name = name
    vb.memory = memory
    vb.cpus = cpus
  end

  # Provision with general playbook (Step 3)
  vm.vm.provision "ansible" do |ansible|
    ansible.compatibility_mode = "2.0"
    ansible.playbook = GENERAL_PLAYBOOK_PATH
    ansible.vault_password_file = "~/.vault_pass.txt"
    ansible.extra_vars = {
      worker_count: NUM_WORKERS
    }
  end

  # Provision with node-specific playbook (Step 3)
  # (ctrl.yaml for controller, node.yaml for worker nodes)
  vm.vm.provision "ansible" do |ansible|
    ansible.compatibility_mode = "2.0"
    ansible.playbook = playbook
    ansible.vault_password_file = "~/.vault_pass.txt"
    ansible.extra_vars = {
      api_server_advertised_address: "192.168.56.100",
    }
  end
end


# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # Define the control node
  config.vm.define "ctrl" do |ctrl|
    vm_setup(
      ctrl,
      name: "ctrl",
      ip: CTRL_IP,
      cpus: CTRL_CPU,
      memory: CTRL_MEMORY,
      playbook: CTRL_PLAYBOOK_PATH,
    )
  end

  # Define the worker nodes 
  (1..NUM_WORKERS).each do |num|
    config.vm.define "node-#{num}" do |node|
      vm_setup(
        node,
        name: "node-#{num}",
        ip: "#{WORKER_IP_BASE}#{100 + num}",
        cpus: WORKER_CPU,
        memory: WORKER_MEMORY,
        playbook: NODE_PLAYBOOK_PATH,
      )
    end

    # Generate inventory.cfg
    config.trigger.before :up do |trigger|
      trigger.ruby do
        file = File.join(File.dirname(__FILE__), "provisioning", "inventory.cfg")
        File.open(file, 'w') do |f|
          f.puts "[ctrl]"
          f.puts "#{CTRL_IP}"
          f.puts ""          
          f.puts "[nodes]"
          WORKER_IP_LIST.each do |ip|
            f.puts "#{ip}"
          end
        end
      end
    end

  end
  

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Disable the default share of the current code directory. Doing this
  # provides improved isolation between the vagrant box and your host
  # by making sure your Vagrantfile isn't accessible to the vagrant box.
  # If you use this you may want to enable additional shared subfolders as
  # shown above.
  # config.vm.synced_folder ".", "/vagrant", disabled: true

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

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL


end
