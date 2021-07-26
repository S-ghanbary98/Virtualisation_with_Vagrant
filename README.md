# Setting Up Development ENV
## Installation of Vagrant, Virtual box and Ruby
![](MicrosoftTeams-image.png)




### Vagrant Commands

- `vagrant up` to launch the vm
- `vagrant destroy` to delete everything
- `vagrant reload` to reload any new instructions in out `Vagrantfile`
- `Vagrant halt` to pause the vm
- `vagrant` to see all commands
-`vagrant init` to initialize new environment by creating a default `vagrantfile`
  
### Vagrant
```python
Usage: vagrant [options] <command> [<args>]

    -h, --help                       Print this help.

Common commands:
     autocomplete    manages autocomplete installation on host
     box             manages boxes: installation, removal, etc.
     cloud           manages everything related to Vagrant Cloud
     destroy         stops and deletes all traces of the vagrant machine
     global-status   outputs status Vagrant environments for this user
     halt            stops the vagrant machine
     help            shows the help for a subcommand
     init            initializes a new Vagrant environment by creating a Vagrantfile
     login
     package         packages a running vagrant environment into a box
     plugin          manages plugins: install, uninstall, update, etc.
     port            displays information about guest port mappings
     powershell      connects to machine via powershell remoting
     provision       provisions the vagrant machine
     push            deploys code in this environment to a configured destination
     rdp             connects to machine via RDP
     reload          restarts vagrant machine, loads new Vagrantfile configuration
     resume          resume a suspended vagrant machine
     snapshot        manages snapshots: saving, restoring, etc.
     ssh             connects to machine via SSH
     ssh-config      outputs OpenSSH valid configuration to connect to the machine
     status          outputs status of the vagrant machine
     suspend         suspends the machine
     up              starts and provisions the vagrant environment
     upload          upload to machine via communicator
     validate        validates the Vagrantfile
     vbguest         plugin: vagrant-vbguest: install VirtualBox Guest Additions to the machine
     version         prints current and latest Vagrant version
     winrm           executes commands on a machine via WinRM
     winrm-config    outputs WinRM configuration to connect to the machine

For help on any individual command run `vagrant COMMAND -h`

Additional subcommands are available, but are either more advanced
or not commonly used. To see all subcommands, run the command
`vagrant list-commands`.
        --[no-]color                 Enable or disable color output
        --machine-readable           Enable machine readable output
    -v, --version                    Display Vagrant version
        --debug                      Enable debug output
        --timestamp                  Enable timestamps on log output
        --debug-timestamp            Enable debug output with timestamps
        --no-tty                     Enable non-interactive output

```



- Let's `ssh` into our vm and launch nginx web-server.
- use `apt-get` package manager in linux.
- `apt-get` is used to install and uninstall any packages needed.
- To use the command `admin` mode we use `sudo` before the command.
- `sudo apt-get upgrade -y`
- `sudo apt-get update -y`
- ping www.bbc.co.uk
- `sudo apt-get install name_of_the_package`
- To work in an `admin mode` at all times (not recommended). `sudo -su` to land in admin mode.
- we will install nginx page in host machines browser.

- Let's automate the tasks that we did manually.
- Create a file called `provision.sh`
- `sudo apt-get update -y`
- `sudo apt-get upgrade -y`
- `sudo apt-get install nginx`
- `systemctl status nginx`


## Automating Virtual Machine Commands

- To save time we are able to run all `sudo` command by pre-writing them in a file `provision.sh`

- To run provision.sh we need to give file permission and make this file executable.
- To change permissions we us `chmod` with required permission then file name.
- `chmod +x provision.sh`
- `sudo ./provision.sh` to run the file.
```provision.sh
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install nginx -y
```
## Sync Folders From Host Machine to Virtual Machine
- We can sync all folders from our host machine to our virtual machine using the command `config.vm.synced_folder ".", "/home/vagrant/"`
- The dot '.' instructs that all folders within root be available in the Virtual Machine.


## Carrying out integration Tests and Running app.js

- The goal is to run app.js however app.js requires some dependencies such as `nodejs`, `npm` ect.
- We want to install these but also monitor if they have passed the test. Once all tests have passed we will be able to run app.js

### Integration Test


- We carry out integreation tests using a sepearate terminal.
- Enter tests directory using `cd environment/spec-tests`
- Run the tests using `rake spec`. This will show all failures and test outcomes which we need to install.
- We can add the tests to the `provision.sh` to automate the tests.


- To pass the tests the code below must be typed in the terminal of the virtual machine

```python
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install nodejs -y 
sudo apt-get install npm -y
sudo apt-get install python-software-properties -y
sudo npm install pm2 -g -y
```

### Running app.js

-  To run the app we must go into the directory of the app using `cd environment/app`
- then we type the following.

```python
npm install
node app.js
```


- The app can now be seen on any browser at `192.168.11.100:3000`


 ## Reverse Proxy

- We can run a reverse proxy so that we can run app.js, by changing the server setting of nginx.
- This will result in us being able to run the app.js in the browser without the :3000 extension.
- We have change the directory by typing the following in the virtual machine terminal.

```
cd /etc/nginx/sites-available
sudo rm -rf default
sudo echo "server{
       listen 80;
       server_name _;
       location / {
       proxy_pass http://192.168.10.100:3000;
       proxy_http_version 1.1;
       proxy_set_header Upgrade \$http_upgrade;
       proxy_set_header Connection 'upgrade';
       proxy_set_header Host \$host;
       proxy_cache_bypass \$http_upgrade;
       }
}" >> default
sudo nginx -t
```
- The following will change the default in nginx and the last line will check for syntax errors.
- We can now run app.js and see it visible in the browser via `102.268.10.100`.



## Running and Connecting Multiple Machines
- We can run multiple Machines using on vagrant file by defining them through via `config.vm.define "name" do |name|`.
- The following below is the vagrant file for two machines named "app" and "db"
- Both have automated processes that are run via syntax `_name.vm.provision` 

``` 
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
```

### db Machine Configuration

- `provisiondb.sh` is used to install mongodb on the "db" machine and all other dependencies. provisiondb.sh can be seen below.
- This is to prepare 'db' machine to connect to 'app'
- In short an update and upgrade is c

```

!#/bash/bin

sudo apt-get update -y
sudo apt-get upgrade -y

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D68FA50FEA312927
echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
sudo apt-get update -y
sudo apt-get upgrade -y

sudo apt-get install mongodb-org=3.2.20 -y
sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20

cd /etc
rm -rf mongod.conf
echo "# mongod.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/
# Where and how to store data.
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log
# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0
#processManagement:
#security:
#operationProfiling:
#replication:
#sharding:
## Enterprise-Only Options:
#auditLog:
#snmp:
" >> mongod.conf
sudo systemctl restart mongod
sudo systemctl status mongod
```

### app Machine Configuration











