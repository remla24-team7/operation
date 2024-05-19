Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-24.04"
  config.vm.box_version = "202404.26.0"

  # config.vm.synced_folder "/mnt/c/remla/vagrant-shared", "/vagrant"

  NETWORK_PREFIX = "192.168.56"
  OFFSET = 110

  CONTROLLER_IP = "#{NETWORK_PREFIX}.#{OFFSET}"
  config.vm.define "controller" do |controller|
    HOSTNAME = "controller"
    controller.vm.network "private_network", ip: CONTROLLER_IP
    # controller.vm.network :forwarded_port, guest: 22, host: 2200, host_ip: "0.0.0.0", id: "ssh"
    controller.vm.hostname = HOSTNAME
    controller.vm.provider "virtualbox" do |vb|
      vb.cpus = 1
      vb.memory = 4096
    end
    controller.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "ansible/playbook.yml"
      ansible.extra_vars = {
        controller_ip: CONTROLLER_IP,
        node_ip: CONTROLLER_IP
      }
      ansible.groups = {
        "controller" => [HOSTNAME]
      }
    end
  end


  (1..2).each do |id|
    config.vm.define "node#{id}" do |node|
      HOSTNAME = "node#{id}"
      NODE_IP = "#{NETWORK_PREFIX}.#{OFFSET + id}"
      node.vm.network "private_network", ip: NODE_IP
      # node.vm.network :forwarded_port, guest: 22, host: 2200 + id, host_ip: "0.0.0.0", id: "ssh"
      node.vm.hostname = HOSTNAME
      node.vm.provider "virtualbox" do |vb|
        vb.cpus = 2
        vb.memory = 6144
      end
      node.vm.provision "ansible" do |ansible|
        ansible.compatibility_mode = "2.0"
        ansible.playbook = "ansible/playbook.yml"
        ansible.extra_vars = {
          controller_ip: CONTROLLER_IP,
          node_ip: NODE_IP
        }
        ansible.groups = {
          "node" => [HOSTNAME]
        }
      end
    end
  end
end
