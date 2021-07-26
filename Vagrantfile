

Vagrant.configure("2") do |config|


 config.vm.define "app" do |app|
    app.vm.box = "ubuntu/xenial64"
    app.vm.synced_folder ".", "/home/vagrant/environment"
    app.vm.provision "shell", path: "provision.sh", env: {'DB_HOST' => 'mongodb://192.168.10.150:27017/posts'}
    app.vm.network "private_network", ip: "192.168.10.100"

 end

  config.vm.define "db" do |db|
    db.vm.box = "ubuntu/xenial64"
    db.vm.network "private_network", ip: "192.168.10.150"
    db.vm.provision "shell", path: "provisiondb.sh"
 end


end
