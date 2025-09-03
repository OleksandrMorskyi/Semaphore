Vagrant.configure("2") do |config|


      # Ansible-Controller (Ubuntu 22.04)
  config.vm.define "ansible" do |app|
    app.vm.box = "bento/ubuntu-22.04"
    app.vm.hostname = "ansible"
    app.vm.network "private_network", ip: "192.168.221.167"
    app.vm.provider "vmware_desktop" do |vm|
      vm.memory = "2024"
      vm.cpus = 2
    end
  end
  
    # Application Server 1 (Ubuntu 22.04)
  config.vm.define "client1" do |app|
    app.vm.box = "bento/ubuntu-22.04"
    app.vm.hostname = "client1"
    app.vm.network "private_network", ip: "192.168.221.168"
    app.vm.provider "vmware_desktop" do |vm|
      vm.memory = "1024"
      vm.cpus = 2
    end
  end

  # Application Server 2 (Ubuntu 22.04)
  config.vm.define "client2" do |app|
    app.vm.box = "bento/ubuntu-22.04"
    app.vm.hostname = "client2"
    app.vm.network "private_network", ip: "192.168.221.169"
    app.vm.provider "vmware_desktop" do |vm|
      vm.memory = "1024"
      vm.cpus = 2
    end
  end

  # Application Server 3 (Ubuntu 22.04)
  config.vm.define "client3" do |db|
    db.vm.box = "bento/ubuntu-22.04"
    db.vm.hostname = "client3"
    db.vm.network "private_network", ip: "192.168.221.173"
    db.vm.provider "vmware_desktop" do |vm|
      vm.memory = "1024"
      vm.cpus = 2
    end
  end

end