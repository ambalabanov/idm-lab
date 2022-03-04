# -*- mode: ruby -*-
# vi: set ft=ruby :

NODES_HOSTNAME = ["ansible","ipa1","ipa2","logstash"]
NODES_FQDN = ["ansible.idm.lab","ipa1.idm.lab","ipa2.idm.lab","logstash.idm.lab"]

Vagrant.configure("2") do |config|
  config.vm.provider "libvirt" do |libvirt|
    libvirt.qemu_use_session = false
  end
  (0..3).each do |i|
    config.vm.define NODES_HOSTNAME[i] do |node|
      node.vm.box = "ambalabanov/rhel"
      # node.vm.box_version = "2.0.0"
      node.vm.box_download_insecure =true
      node.vm.hostname = NODES_FQDN[i]
      node.vm.network "private_network", ip: "192.168.55.1#{i}"
      node.vm.synced_folder ".", "/vagrant", type: "rsync"
      node.vm.provider "parallels" do |prl|
        prl.check_guest_tools = true
        prl.update_guest_tools = true
        prl.name = NODES_HOSTNAME[i]
        prl.memory = 3072
        prl.cpus = 2
      end
      node.vm.provider "libvirt" do |libvirt|
        libvirt.memory = 3072
        libvirt.cpus = 2
        libvirt.driver = "kvm"
      end
      node.vm.provision "hosts", type: "shell" do |shell|
        shell.privileged = true
        shell.inline = <<-SHELL
          cp -f /vagrant/hosts /etc/hosts
        SHELL
      end
      node.vm.provision "00-subscription", type: "ansible" do |ansible|
        ansible.playbook = "playbooks/00-subscription.yml"
        ansible.ask_vault_pass = true
      end
      if node.vm.hostname == "ansible.idm.lab"
        node.vm.provision "05-ansible", type: "ansible" do |ansible|
          ansible.playbook = "playbooks/05-ansible.yml"
        end
        node.vm.provision  "ssh", type: "shell" do |shell|
          shell.privileged = false
          shell.inline = <<-SHELL
            ssh-keygen -f ~/.ssh/id_rsa -N ""
            ssh-keyscan -t rsa -H ansible.idm.lab >> ~/.ssh/known_hosts
            ssh-keyscan -t rsa -H ipa1.idm.lab >> ~/.ssh/known_hosts
            ssh-keyscan -t rsa -H ipa2.idm.lab >> ~/.ssh/known_hosts
            ssh-keyscan -t rsa -H logstash.idm.lab >> ~/.ssh/known_hosts
            sshpass -p vagrant ssh-copy-id ansible.idm.lab
            sshpass -p vagrant ssh-copy-id ipa1.idm.lab
            sshpass -p vagrant ssh-copy-id ipa2.idm.lab
            sshpass -p vagrant ssh-copy-id logstash.idm.lab
          SHELL
        end
      end
      node.trigger.before :destroy do |trigger|
        trigger.name = "Unregister RHN Guest"
        trigger.warn = "Unregister"
        trigger.run_remote = {inline: <<-SHELL
        subscription-manager remove --all
        subscription-manager unregister
        subscription-manager clean
        SHELL
        }
      end
    end
  end  
end