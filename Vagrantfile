# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = "512"
    vb.cpus = 2 # not less than the required 2
  end

  config.vm.define "k8s-master01" do |k8scluster|
      k8scluster.vm.box = "bento/ubuntu-19.04"
      k8scluster.vm.hostname = "k8s-master01"
      k8scluster.vm.network "private_network", ip: "192.168.50.10"
      k8scluster.vm.provider "virtualbox" do |vb|
          vb.name = "k8s-master01"
          vb.memory = "1024"
      end
      k8scluster.vm.provision "ansible_local" do |ansible|
        ansible.playbook = "provisioning/deploy.yml"
        ansible.become = true
        ansible.compatibility_mode = "2.0"
        ansible.version = "2.9.4" # ubuntu-19.04s
        ansible.extra_vars = {
                master_ip: "192.168.50.10",
            }
      end
      k8scluster.vm.provision "shell", inline: <<-SHELL
            hostnamectl status
      SHELL
    end

    config.vm.define "worker01" do |k8scluster|
        k8scluster.vm.box = "bento/ubuntu-16.04"
        k8scluster.vm.hostname = "worker01"
        k8scluster.vm.network "private_network", ip: "192.168.50.11"
        k8scluster.vm.provider "virtualbox" do |vb|
            vb.name = "worker01"
            # vb.memory = "768"
        end
        k8scluster.vm.provision "ansible_local" do |ansible|
          ansible.playbook = "provisioning/deploy.yml"
          ansible.become = true
          ansible.compatibility_mode = "2.0"
          ansible.version = "2.9.4" # ubuntu-16.04
          ansible.extra_vars = {
                  worker1_ip: "192.168.50.11",
              }
        end
        k8scluster.vm.provision "shell", inline: <<-SHELL
              hostnamectl status
        SHELL
      end


      config.vm.define "worker02" do |k8scluster|
          k8scluster.vm.box = "bento/ubuntu-18.10"
          k8scluster.vm.hostname = "worker02"
          k8scluster.vm.network "private_network", ip: "192.168.50.12"
          k8scluster.vm.provider "virtualbox" do |vb|
              vb.name = "worker02"
              # vb.memory = "768"
          end
          k8scluster.vm.provision "ansible_local" do |ansible|
            ansible.playbook = "provisioning/deploy.yml"
            ansible.become = true
            ansible.compatibility_mode = "2.0"
            ansible.version = "2.8.2" # ubuntu-18.10
            ansible.extra_vars = {
                    worker2_ip: "192.168.50.12",
                }
          end
          k8scluster.vm.provision "shell", inline: <<-SHELL
                hostnamectl status
          SHELL
        end

        config.vm.define "remotecontrol01" do |k8scluster|
            k8scluster.vm.box = "bento/centos-7.7" # OK
            k8scluster.vm.hostname = "remotecontrol01"
            k8scluster.vm.network "private_network", ip: "192.168.50.13"
            k8scluster.vm.provider "virtualbox" do |vb|
                vb.name = "remotecontrol01"
                # vb.memory = "512"
            end
            k8scluster.vm.provision "ansible_local" do |ansible|
              ansible.playbook = "provisioning/deploy.yml"
              ansible.become = true
              ansible.compatibility_mode = "2.0"
              ansible.version = "2.9.2" # ubuntu-16.04
              # ansible.extra_vars = {
                  #     node_ip: "192.168.50.12",
                  # }
            end
            k8scluster.vm.provision "shell", inline: <<-SHELL
                  hostnamectl status
            SHELL
          end
end
