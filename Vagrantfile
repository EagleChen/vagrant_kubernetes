# -*- mode: ruby -*-
VAGRANTFILE_API_VERSION = "2"

# remember to recheck /etc/hosts for each host, might need to modify it

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "centos7"
  config.vm.url = "https://atlas.hashicorp.com/relativkreativ/boxes/centos-7-minimal/versions/1.0.3/providers/virtualbox.box"

  config.vm.provider "virtualbox" do |v|
    v.customize ["modifyvm", :id, "--memory", 1024]
  end

  minions = 2

  # make sure docker group is added !!!!
  script = <<-SCRIPT
    yum makecache fast
    yum install -y docker
    curl https://raw.githubusercontent.com/docker/docker/master/contrib/init/systemd/docker.service -o /etc/systemd/system/docker.service
    curl https://raw.githubusercontent.com/docker/docker/master/contrib/init/systemd/docker.socket -o /etc/systemd/system/docker.socket
    groupadd docker
    yum update -y device-mapper-libs

    rpm -i /vagrant/kubernetes-0.14.2-0.1.git2719194.fc23.x86_64.rpm

    systemctl disable iptables-services firewalld
    systemctl stop iptables-services firewalld

    cp /vagrant/ku_conf /etc/kubernetes/config
  SCRIPT

  script << "echo '192.168.1.3 master' >> /etc/hosts"
  minions.times do |index|
    script << "echo '192.168.1.#{4+index} minion-#{index}' >> /etc/hosts"
  end

  config.vm.provision "shell", inline: script

  config.vm.define "master" do |master|
    master.vm.hostname = "master"
    master.vm.network :private_network, ip: "192.168.1.3"

    master_script = <<-SCRIPT
      yum install -y etcd
      cp /vagrant/etcd.service /etc/systemd/system/
      cp /vagrant/ku_master_api /etc/kubernetes/apiserver
      cp /vagrant/ku_controller_manager /etc/kubernetes/controller-manager

      for SERVICES in etcd kube-apiserver kube-controller-manager kube-scheduler; do
        systemctl restart $SERVICES
        systemctl enable $SERVICES
        systemctl status $SERVICES
      done
    SCRIPT

    config.vm.provision "shell", inline: script + "\n" + master_script
  end

  minions.times do |index|
    minion_id = "minion-#{index}"
    config.vm.define minion_id do |minion|
      minion.vm.hostname = minion_id
      minion.vm.network :private_network, ip: "192.168.1.#{4+index}"

      node_script = <<-SCRIPT
        cp /vagrant/node_kubelet /etc/kubernetes/kubelet
        for SERVICES in kube-proxy kubelet docker; do
          systemctl restart $SERVICES
          systemctl enable $SERVICES
          systemctl status $SERVICES
        done
      SCRIPT

      config.vm.provision "shell", inline: script + "\n" + node_script
    end
  end
end
