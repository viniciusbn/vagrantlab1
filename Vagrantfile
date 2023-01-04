Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  config.vm.provider "virtualbox" do |virtualbox|
    virtualbox.memory = 512
    virtualbox.cpus = 1
  end

  # config.vm.define "mysqldb" do |mysql|
  #   $script_mysql = <<-SCRIPT
  #     apt-get update && \
  #     apt-get install -y mysql-server-5.7 && \
  #     mysql -e "create user 'phpuser'@'%' identified by 'pass';"
  #   SCRIPT
  #   #mysql.vm.network "forwarded_port", id: "MySQL", guest: 3306, host: 3306, host_ip: "127.0.0.1", protocol: "tcp"
  #   mysql.vm.network "private_network", ip: "192.168.50.20"
  #   mysql.vm.synced_folder "./configs", "/configs"
  #   #mysql.vm.synced_folder ".", "/vagrant", disabled: "true"
  #   mysql.vm.provision "shell", inline: $script_mysql
  #   mysql.vm.provision "shell", inline: "cat /configs/mysqld.cnf > /etc/mysql/mysql.conf.d/mysqld.cnf"
  #   mysql.vm.provision "shell", inline: "service mysql restart"
  #   mysql.vm.provision "shell", inline: "cat /configs/id_bionic.pub >> .ssh/authorized_keys"
  # end

  config.vm.define "phpweb" do |phpweb|
    phpweb.vm.provider "virtualbox" do |virtualbox|
      virtualbox.memory = 1024
      virtualbox.cpus = 2
      virtualbox.name = "ubuntu_bionirc_php7"
    end
    phpweb.vm.network "forwarded_port", id: "httpd", guest: 80, host: 8080, host_ip: "127.0.0.1", protocol: "tcp"
    phpweb.vm.network "forwarded_port", id: "httpdPHP", guest: 8888, host: 8888, host_ip: "127.0.0.1", protocol: "tcp"
    phpweb.vm.network "private_network", ip: "192.168.50.10"
    phpweb.vm.synced_folder "./configs", "/configs"
    #phpweb.vm.synced_folder ".", "/vagrant", disabled: "true"
    phpweb.vm.provision "shell",
      inline: "apt update && apt install puppet -y"
    phpweb.vm.provision "puppet" do |puppet|
      puppet.manifests_path = "./configs/manifests"
      puppet.manifest_file = "phpweb.pp"
    end
    phpweb.vm.provision "shell", inline: "cat /configs/id_bionic.pub >> .ssh/authorized_keys"
  end

  config.vm.define "mysqlserver" do |mysqlserver|
    mysqlserver.vm.network "private_network", ip: "192.168.50.30"
    mysqlserver.vm.synced_folder "./configs", "/configs"
    mysqlserver.vm.provision "shell", inline: "cat /configs/id_bionic.pub >> .ssh/authorized_keys"
  end

  config.vm.define "ansible" do |ansible|
    $script_ansyble = <<-SCRIPT
      apt update && \
      apt install python3-distutils python3-apt -y && \
      curl https://bootstrap.pypa.io/pip/3.6/get-pip.py -o get-pip.py && \
      python3 get-pip.py && \
      python3 -m pip install ansible
    SCRIPT
    ansible.vm.network "private_network", ip: "192.168.50.40"
    ansible.vm.synced_folder "./configs", "/configs"
    ansible.vm.provision "shell", inline: $script_ansyble
    ansible.vm.provision "shell", inline: "cat /configs/id_bionic.pub >> .ssh/authorized_keys"
    ansible.vm.provision "shell",
      inline: "cp /configs/id_bionic /home/vagrant && \
              chmod 600 /home/vagrant/id_bionic && \
              chown vagrant:vagrant /home/vagrant/id_bionic"
    ansible.vm.provision "shell", inline: "ansible-playbook -i /configs/ansible/hosts /configs/ansible/playbook.yml"
  end

  config.vm.define "memcached" do |memcached|
    memcached.vm.box = "centos/7"
    memcached.vm.provider "virtualbox" do |vb|
        vb.memory = 512
        vb.cpus = 1
        vb.name = "centos7_memcached"
    end
  end
  config.vm.define "dockerhost" do |dockerhost|
  dockerhost.vm.provider "virtualbox" do |virtualbox|
      virtualbox.memory = 512
      virtualbox.cpus = 1
      virtualbox.name = "ubuntu_dockerhost"
  end
  $script_docker = <<-SCRIPT
    apt update && \
    apt install -y \
        ca-certificates \
        curl \
        gnupg \
        lsb-release && \
    mkdir -p /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg && \
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null && \
    chmod a+r /etc/apt/keyrings/docker.gpg && \
    apt update && \
    apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
  SCRIPT
  dockerhost.vm.network "private_network", ip: "192.168.50.50"
  dockerhost.vm.synced_folder "./configs", "/configs"
  dockerhost.vm.provision "shell", inline: $script_docker
  dockerhost.vm.provision "shell", inline: "cat /configs/id_bionic.pub >> .ssh/authorized_keys"
  end
end