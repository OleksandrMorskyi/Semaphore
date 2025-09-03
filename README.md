# 🖥️ Ansible Vagrant-Setup mit VMware

Dieses Setup erstellt eine vollständige Testumgebung mit:

- ✅ 1x Ansible-Controller (`Ansible`)
- ✅ 3x Ubuntu-Clients (`Client1`, `Client2`, `Client3`)
- 🧠 Alle über ein privates Netzwerk verbunden
- 🛠️ Kompatibel mit VMware Workstation / Fusion

---

## 📁 Vagrantfile

```ruby
Vagrant.configure("2") do |config|

  # Ansible-Controller (Ubuntu 22.04)
  config.vm.define "Ansible" do |app|
    app.vm.box = "bento/ubuntu-22.04"
    app.vm.hostname = "Ansible"
    app.vm.network "private_network", ip: "192.168.221.170"
    app.vm.provider "vmware_desktop" do |vm|
      vm.memory = "1024"
      vm.cpus = 2
    end
  end

  # Client 1
  config.vm.define "Client1" do |app|
    app.vm.box = "bento/ubuntu-22.04"
    app.vm.hostname = "Client1"
    app.vm.network "private_network", ip: "192.168.221.171"
    app.vm.provider "vmware_desktop" do |vm|
      vm.memory = "1024"
      vm.cpus = 2
    end
  end

  # Client 2
  config.vm.define "Client2" do |app|
    app.vm.box = "bento/ubuntu-22.04"
    app.vm.hostname = "Client2"
    app.vm.network "private_network", ip: "192.168.221.172"
    app.vm.provider "vmware_desktop" do |vm|
      vm.memory = "1024"
      vm.cpus = 2
    end
  end

  # Client 3 (z. B. Datenbank)
  config.vm.define "Client3" do |db|
    db.vm.box = "bento/ubuntu-22.04"
    db.vm.hostname = "client3"
    db.vm.network "private_network", ip: "192.168.221.173"
    db.vm.provider "vmware_desktop" do |vm|
      vm.memory = "1024"
      vm.cpus = 2
    end
  end

end

🚀 Verwendung
Terminal öffnen im Ordner mit dem Vagrantfile
Starte alle VMs:
bash
vagrant up
Melde dich am Controller an:
bash
vagrant ssh Ansible
⚙️ Auf dem Controller (VM "Ansible")
Ansible installieren
bash
sudo apt update && sudo apt install ansible sshpass -y
SSH-Zugriff vorbereiten
bash
ssh-keygen -t rsa
ssh-copy-id vagrant@192.168.221.171
ssh-copy-id vagrant@192.168.221.172
ssh-copy-id vagrant@192.168.221.173
(ggf. vorher sshpass installieren)
📋 Ansible-Inventar erstellen (inventory.ini)
ini
[clients]
client1 ansible_host=192.168.221.171 ansible_user=vagrant
client2 ansible_host=192.168.221.172 ansible_user=vagrant
client3 ansible_host=192.168.221.173 ansible_user=vagrant
Verbindung testen:
bash
ansible -i inventory.ini clients -m ping
```
