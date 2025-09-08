# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # Definizione della VM Jenkins
  config.vm.define "jenkins" do |jenkins|
    jenkins.vm.box = "generic/rocky9"  # Immagine base Rocky Linux 9
    jenkins.vm.hostname = "jenkins-lab"  # Nome host della VM
    jenkins.vm.network "private_network", ip: "192.168.56.10"  # Rete privata per accesso da host

    # Configurazione risorse con provider libvirt
    jenkins.vm.provider :libvirt do |lv|
      lv.memory = 2048  # RAM assegnata (2 GB)
      lv.cpus = 2       # CPU assegnate
      lv.driver = "kvm" # Tipo di virtualizzazione
    end

    jenkins.vm.synced_folder ".", "/vagrant", type: "rsync"

    # Provisioning: installa Docker, Jenkins master, agent.
    jenkins.vm.provision "ansible" do |ansible|
      ansible.playbook = "main.yml"
      ansible.inventory_path = "ansible/inventory.ini"
    end
  end
end