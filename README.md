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

Ось повний README.md — покрокова інструкція, як налаштувати Ansible, Semaphore та керувати 4-ма Vagrant-машинами з локального контролера. Структуровано, зрозуміло, і готово до продакшну 🚀
markdown

# 🚀 Infrastructure Automation with Ansible & Semaphore

Цей проєкт дозволяє автоматизувати керування 4-ма віртуальними машинами (VM) через Ansible, використовуючи Semaphore як інтерфейс для запуску задач. Контролер — локальна машина (Mac), яка підключається до VM через SSH.

---

## 📦 Вимоги

- macOS або Linux
- Встановлений [Vagrant](https://www.vagrantup.com/)
- Встановлений [VMware Desktop](https://www.vmware.com/)
- Встановлений [Docker](https://www.docker.com/)
- Встановлений [Ansible](https://docs.ansible.com/)
- Semaphore (в контейнері)

---

## 🧱 Структура VM (Vagrantfile)

```ruby
config.vm.define "ansible" do |app|
  app.vm.network "private_network", ip: "192.168.221.167"
end

config.vm.define "client1" do |app|
  app.vm.network "private_network", ip: "192.168.221.168"
end

config.vm.define "client2" do |app|
  app.vm.network "private_network", ip: "192.168.221.169"
end

config.vm.define "client3" do |app|
  app.vm.network "private_network", ip: "192.168.221.173"
end
🔐 Налаштування SSH-доступу
Створи SSH-ключ:
bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/semaphore_key
Скопіюй публічний ключ на всі VM:
bash
ssh-copy-id -i ~/.ssh/semaphore_key.pub vagrant@192.168.221.167
ssh-copy-id -i ~/.ssh/semaphore_key.pub vagrant@192.168.221.168
ssh-copy-id -i ~/.ssh/semaphore_key.pub vagrant@192.168.221.169
ssh-copy-id -i ~/.ssh/semaphore_key.pub vagrant@192.168.221.173
📁 Створення inventory.ini
ini
[ansible]
192.168.221.167 ansible_user=vagrant ansible_ssh_private_key_file=~/.ssh/semaphore_key ansible_ssh_common_args='-o IdentitiesOnly=yes'

[client1]
192.168.221.168 ansible_user=vagrant ansible_ssh_private_key_file=~/.ssh/semaphore_key ansible_ssh_common_args='-o IdentitiesOnly=yes'

[client2]
192.168.221.169 ansible_user=vagrant ansible_ssh_private_key_file=~/.ssh/semaphore_key ansible_ssh_common_args='-o IdentitiesOnly=yes'

[client3]
192.168.221.173 ansible_user=vagrant ansible_ssh_private_key_file=~/.ssh/semaphore_key ansible_ssh_common_args='-o IdentitiesOnly=yes'
⚙️ Створення ansible.cfg
ini
[defaults]
host_key_checking = False
private_key_file = ~/.ssh/semaphore_key
inventory = ./inventory.ini

[ssh_connection]
ssh_args = -o IdentitiesOnly=yes


🧪 Тестування з'єднання
bash
ansible all -m ping
Очікуваний результат:
json
192.168.221.168 | SUCCESS => { "ping": "pong" }

🧰 Приклад playbook.yml
yaml
- name: Update all servers
  hosts: all
  become: true
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
Запуск:
bash
ansible-playbook playbook.yml

🐳 Запуск Semaphore у Docker
bash
docker run -d \
  --name semaphore \
  -p 3000:3000 \
  -v ~/.ssh:/home/semaphore/.ssh \
  -v $(pwd):/ansible \
  semaphoreui/semaphore:latest

🧠 Налаштування Semaphore
Відкрий http://localhost:3000
Створи проект
Додай Environment:
SSH-ключ: ~/.ssh/semaphore_key
Inventory: inventory.ini

Створи Task:
bash
ansible-playbook playbook.yml

✅ Готово!
Тепер ти можеш керувати всіма VM через Ansible, запускати задачі через Semaphore, і масштабувати свою інфраструктуру як справжній DevOps-інженер 💼🔥
📚 Додатково
Ansible Docs
Semaphore Docs
Vagrant Networking
```

########################################################

Hier ist dein erweitertes README.md – optimiert für GitHub, mit Badges, klarer Struktur und CI/CD-Vorbereitung. Du kannst es direkt in dein Repository packen und glänzen wie ein Profi 💎
markdown

# 🚀 Infrastructure Automation with Ansible & Semaphore

![License](https://img.shields.io/badge/license-MIT-green)
![Status](https://img.shields.io/badge/status-active-brightgreen)
![Made with](https://img.shields.io/badge/made%20with-Ansible-blue)
![Platform](https://img.shields.io/badge/platform-Vagrant%20%26%20Docker-lightgrey)

Automatisiere deine Infrastruktur mit Ansible und Semaphore – verwalte vier Vagrant-VMs über eine zentrale Steuerung auf deinem lokalen Rechner. Dieses Setup ist ideal für DevOps-Training, Testumgebungen oder als Basis für produktionsreife Automatisierung.

---

## 📦 Voraussetzungen

- macOS oder Linux
- [Vagrant](https://www.vagrantup.com/)
- [VMware Desktop](https://www.vmware.com/)
- [Docker](https://www.docker.com/)
- [Ansible](https://docs.ansible.com/)
- SSH-Zugriff auf alle VMs

---

## 🧱 Vagrantfile-Konfiguration

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"

  config.vm.define "ansible" do |app|
    app.vm.network "private_network", ip: "192.168.221.167"
  end

  config.vm.define "client1" do |app|
    app.vm.network "private_network", ip: "192.168.221.168"
  end

  config.vm.define "client2" do |app|
    app.vm.network "private_network", ip: "192.168.221.169"
  end

  config.vm.define "client3" do |app|
    app.vm.network "private_network", ip: "192.168.221.173"
  end
end
🔐 SSH-Konfiguration
SSH-Key generieren:
bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/semaphore_key
Key auf alle VMs kopieren:
bash
ssh-copy-id -i ~/.ssh/semaphore_key.pub vagrant@192.168.221.167
ssh-copy-id -i ~/.ssh/semaphore_key.pub vagrant@192.168.221.168
ssh-copy-id -i ~/.ssh/semaphore_key.pub vagrant@192.168.221.169
ssh-copy-id -i ~/.ssh/semaphore_key.pub vagrant@192.168.221.173
📁 Inventory-Datei (inventory.ini)
ini
[ansible]
192.168.221.167 ansible_user=vagrant ansible_ssh_private_key_file=~/.ssh/semaphore_key

[clients]
192.168.221.168 ansible_user=vagrant ansible_ssh_private_key_file=~/.ssh/semaphore_key
192.168.221.169 ansible_user=vagrant ansible_ssh_private_key_file=~/.ssh/semaphore_key
192.168.221.173 ansible_user=vagrant ansible_ssh_private_key_file=~/.ssh/semaphore_key
⚙️ Ansible-Konfiguration (ansible.cfg)
ini
[defaults]
inventory = ./inventory.ini
host_key_checking = False
private_key_file = ~/.ssh/semaphore_key

[ssh_connection]
ssh_args = -o IdentitiesOnly=yes
🧪 Verbindung testen
bash
ansible all -m ping
Erwartete Ausgabe:
json
192.168.221.168 | SUCCESS => { "ping": "pong" }
📜 Beispiel-Playbook (playbook.yml)
yaml
- name: Update all servers
  hosts: all
  become: true
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
Playbook ausführen:
bash
ansible-playbook playbook.yml
🐳 Semaphore via Docker starten
bash
docker run -d \
  --name semaphore \
  -p 3000:3000 \
  -v ~/.ssh:/home/semaphore/.ssh \
  -v $(pwd):/ansible \
  semaphoreui/semaphore:latest
🧠 Semaphore einrichten
Öffne http://localhost:3000
Neues Projekt erstellen
Environment hinzufügen:
SSH-Key: ~/.ssh/semaphore_key
Inventory: inventory.ini
Task erstellen:
bash
ansible-playbook playbook.yml
🛠️ CI/CD-Vorbereitung (optional)
Falls du GitHub Actions nutzen willst, hier ein Beispiel für .github/workflows/deploy.yml:
yaml
name: Deploy Infrastructure

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install Ansible
        run: sudo apt update && sudo apt install -y ansible
      - name: Run Playbook
        run: ansible-playbook playbook.yml -i inventory.ini
📚 Ressourcen
Ansible Docs
Semaphore Docs
Vagrant Networking
```
