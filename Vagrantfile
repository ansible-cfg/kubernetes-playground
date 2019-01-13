$bootstrap_ansible = <<-SHELL
echo "Installing Ansible..."
sudo apt-get update -y
sudo apt-get install -y software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update -y
sudo apt-get install -y ansible apt-transport-https
SHELL

$restart_kubelet = <<-SHELL
echo "Restarting Kubelet..."
sudo systemctl daemon-reload
sudo systemctl restart kubelet
SHELL

Vagrant.configure(2) do |config|

  (1..3).each do |i|
    config.vm.define "k8s#{i}" do |s|
      s.ssh.forward_agent = true
      s.vm.box = "ubuntu/xenial64"
      s.vm.hostname = "k8s#{i}"
      s.vm.provision :shell,
        inline: $bootstrap_ansible
      if i == 1
        s.vm.provision :shell,
          inline: "PYTHONUNBUFFERED=1 ansible-playbook /vagrant/ansible/k8s-master.yml -c local"
      else
        s.vm.provision :shell,
          inline: "PYTHONUNBUFFERED=1 ansible-playbook /vagrant/ansible/k8s-worker.yml -c local"
      end
      s.vm.provision :shell,
        inline: "echo 'KUBELET_EXTRA_ARGS=--node-ip=172.42.42.#{i+10-1}' | sudo tee /etc/default/kubelet"
      s.vm.provision :shell,
        inline: $restart_kubelet
      s.vm.network "private_network",
        ip: "172.42.42.#{i+10-1}",
        netmask: "255.255.255.0",
        auto_config: true,
        virtualbox__intnet: "k8s-net"
      s.vm.provider "virtualbox" do |v|
        v.name = "k8s#{i}"
        v.cpus = 2
        if i == 1
          v.memory = 2048
        else
          v.memory = 4096
        end
        v.gui = false
      end
    end
  end

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

end
