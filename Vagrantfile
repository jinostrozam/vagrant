# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

    config.vm.provider :libvirt do |kvm|
      kvm.random model: 'random'
      kvm.cpu_mode  = 'host-passthrough'
      kvm.driver    = 'kvm'
      kvm.memory    = 4096
      kvm.cpus      = 2
    end
  
    config.ssh.forward_agent = true
  
    config.vm.define "puppetserver" do |vm1|
  
      vm1.vm.box              = "generic/ubuntu1604"
        vm1.vm.hostname         = "puppetserver"
        vm1.vm.box_check_update = true
   
        vm1.vm.network :private_network,
                        ip: '192.168.150.10',
                        libvirt_netmask: '255.255.255.0',
                        libvirt__network_name: 'new-puppet-lab',
                        autostart: true,
                        libvirt__forward_mode: 'route',
                        libvirt__dhcp_enabled: false 
                   
      vm1.vm.provision "shell", path: "scripts/puppet-server-install.sh"
  
      vm1.vm.provision "shell", :inline => <<-SHELL
        sudo echo -e 'nameserver 8.8.8.8\nnameserver 8.8.4.4\n' > /etc/resolv.conf
        sudo echo "192.168.150.10 puppetserver" | sudo tee -a /etc/hosts
        sudo echo "192.168.150.20 webserver" | sudo tee -a /etc/hosts
        sudo echo "*" | sudo tee -a /etc/puppetlabs/puppet/autosign.conf
        sudo echo "autosign = true" | sudo tee -a /etc/puppetlabs/puppet/puppet.conf
        sudo echo "[main]" | sudo tee -a /etc/puppetlabs/puppet/puppet.conf
        sudo echo -e "certname = puppetserver\nserver = puppetserver\nenvironment = production\nruninterval = 15m" | sudo tee -a /etc/puppetlabs/puppet/puppet.conf
      SHELL
     
#      vm1.vm.provision "shell", :inline => <<-SHELL
#        cd /etc/puppetlabs/code/environments
#        sudo mv production production.orig
#        sudo git clone https://github.com/jinostrozam/puppet.git production
#        cd production
#        sudo git checkout production
#        sudo /opt/puppetlabs/puppet/bin/gem install r10k --no-rdoc --no-ri
#        sudo /opt/puppetlabs/puppet/bin/r10k puppetfile install --verbose
#        sudo /opt/puppetlabs/bin/puppet apply --environment=production /etc/puppetlabs/code/environments/production/manifests/
#      SHELL

    end

    config.ssh.forward_agent = true
  
    config.vm.define "webserver" do |vm2|
  
      vm2.vm.box              = "generic/ubuntu1604"
      vm2.vm.hostname         = "webserver"
      vm2.vm.box_check_update = true
   
      vm2.vm.network :private_network,
                      ip: '192.168.150.20',
                      libvirt_netmask: '255.255.255.0',
                      libvirt__network_name: 'new-puppet-lab',
                      autostart: true,
                      libvirt__forward_mode: 'route',
                      libvirt__dhcp_enabled: false  

                        
      vm2.vm.synced_folder "./puppet", "/puppet-test-dir"
      
      vm2.vm.provision "shell", path: "scripts/puppet-agent-install.sh"
  
      vm2.vm.provision "shell", :inline => <<-SHELL
        sudo echo -e 'nameserver 8.8.8.8\nnameserver 8.8.4.4\n' > /etc/resolv.conf
        sudo echo "192.168.150.10 puppetserver" | sudo tee -a /etc/hosts
        sudo echo "192.168.150.20 webserver" | sudo tee -a /etc/hosts
        sudo echo "autosign = true" | sudo tee -a /etc/puppetlabs/puppet/puppet.conf
        sudo echo "[main]" | sudo tee -a /etc/puppetlabs/puppet/puppet.conf
        sudo echo -e "certname = webserver\nserver = puppetserver\nenvironment = production\nruninterval = 15m" | sudo tee -a /etc/puppetlabs/puppet/puppet.conf        
      SHELL
   
    end

end