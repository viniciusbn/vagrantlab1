Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"

  config.vm.define "mysqldb" do |mysql|
    $script_mysql = <<-SCRIPT
      apt-get update && \
      apt-get install -y mysql-server-5.7 && \
      mysql -e "create user 'phpuser'@'%' identified by 'pass';"
    SCRIPT
    mysql.vm.network "forwarded_port", id: "MySQL", guest: 3306, host: 3306, host_ip: "127.0.0.1", protocol: "tcp"
    mysql.vm.network "private_network", ip: "192.168.50.20"
    mysql.vm.synced_folder "./configs", "/configs"
    #mysql.vm.synced_folder ".", "/vagrant", disabled: "true"
    mysql.vm.provision "shell", inline: $script_mysql
    mysql.vm.provision "shell", inline: "cat /configs/mysqld.cnf > /etc/mysql/mysql.conf.d/mysqld.cnf"
    mysql.vm.provision "shell", inline: "service mysql restart"
  end
  config.vm.define "phpweb" do |phpweb|
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
  end
end