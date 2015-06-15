# -*- mode: ruby -*-
# vi: set ft=ruby :

INSTANCES=2
DOMAIN="example.com"
SUBNET="192.168.33"

Vagrant.configure(2) do |config|
  config.vm.box = "pug/scientific"
  config.vm.provider :virtualbox do |vb|
      vb.memory = "384"
  end

  INSTANCES.times do |i|
      config.vm.define "proxy#{i}".to_sym do |haproxy|
        haproxy.vm.hostname = "proxy#{i}.#{DOMAIN}"
        haproxy.vm.network :private_network, ip: "#{SUBNET}.%d" % ( 10 + i + 1 )
        haproxy.vm.provision "shell", inline: <<-SHELL
            sudo yum install -y keepalived haproxy
            sudo cp /vagrant/haproxy.cfg /etc/haproxy/haproxy.cfg
            sudo cp /vagrant/keepalived.conf /etc/keepalived/keepalived.conf
            sudo service keepalived restart
            sudo service haproxy restart
        SHELL
      end
  end

  config.vm.define "web".to_sym do |web|
    web.vm.hostname = "web.#{DOMAIN}"
    web.vm.network :private_network, ip: "#{SUBNET}.9"
    web.vm.provision "shell", inline: <<-SHELL
        sudo cp /vagrant/nginx-release.repo /etc/yum.repos.d/
        sudo yum install -y nginx arptables_jf

        sudo service nginx start

        sudo cp /vagrant/arptables /etc/sysconfig/arptables
        sudo service arptables_jf start
        sudo chkconfig --level 2345 arptables_jf on
        sudo ip addr add 192.168.33.10/32 dev eth1
    SHELL
  end

end
