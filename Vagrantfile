Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-24.04"
  config.vm.box_version = "202404.26.0"

  # config.vm.synced_folder "/mnt/c/remla/vagrant-shared", "/vagrant"

  NETWORK_PREFIX = "192.168.56"
  OFFSET = 110

  config.vm.define "controller" do |controller|
    controller.vm.network "private_network", ip: "#{NETWORK_PREFIX}.#{OFFSET}"
    # controller.vm.network :forwarded_port, guest: 22, host: 2200, host_ip: "0.0.0.0", id: "ssh"
    controller.vm.hostname = "controller"
    controller.vm.provider "virtualbox" do |vb|
      vb.cpus = 1
      vb.memory = 4096
    end
    controller.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "ansible/playbook.yml"
      ansible.groups = {
        "controller" => ["controller"]
      }
    end
  end


  (1..2).each do |i|
    config.vm.define "node#{i}" do |node|
      node.vm.network "private_network", ip: "#{NETWORK_PREFIX}.#{OFFSET+i}"
      # node.vm.network :forwarded_port, guest: 22, host: 2200+i, host_ip: "0.0.0.0", id: "ssh"
      node.vm.hostname = "node#{i}"
      node.vm.provider "virtualbox" do |vb|
        vb.cpus = 2
        vb.memory = 6144
      end
      node.vm.provision "ansible" do |ansible|
        ansible.compatibility_mode = "2.0"
        ansible.playbook = "ansible/playbook.yml"
        ansible.groups = {
          "node" => ["node#{i}"]
        }
      end
    end
  end
end
