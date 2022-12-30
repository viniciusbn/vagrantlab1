Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"

  config.vm.define "mysqldb" do |mysql|
    mysql.vm.synced_folder "./configs", "/configs"
    mysql.vm.synced_folder ".", "/vagrant", disabled: "true"
    mysql.vm.provision "shell", inline: "echo hello, World"
  end
  config.vm.define "phpweb" do |phpweb|
    phpweb.vm.network "forwarded_port", id: "httpd", guest: 80, host: 8080, host_ip: "127.0.0.1", protocol: "tcp"
    phpweb.vm.network "forwarded_port", id: "httpdPHP", guest: 8888, host: 8888, host_ip: "127.0.0.1", protocol: "tcp"
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