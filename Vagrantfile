# -*- mode: ruby -*-
# vi: set ft=ruby :
# v2.0
# Author: Cooper Hanson
# Comment:  Auto deploy a LAMP application

sql_pwd = 'sql_pwd'
web_ip = '192.168.56.10'
db_ip = '192.168.56.11'

Vagrant.configure("2") do |config|
    
    #define a  do block for the web server 
    config.vm.define "web" do |web|
  
     #use the ubuntu/bionic64 VM template - it is the official Ubuntu one
     #if you want to use Ubuntu 20.04, replace bionic64 with focal64 
     web.vm.box = "ubuntu/focal64"
 
     #setting up the networking such that the web server
     #will have two network adapters:  a NAT network with a port forward
     #to allow the outside world to connect to the server
     #and also a private "host only" network to allow the web server
     #to talk to the db server 
     web.vm.network "forwarded_port", guest: 80, host: 80
     web.vm.network "private_network", ip: web_ip
        
     #copy any needed files out to the vm

     #set up a provision block to install and configure the apache/php server

     web.vm.provision "shell", inline: <<-SHELL
	   apt update
	   apt upgrade -y
	   apt install -y apache2 ufw
	   ufw enable
	   ufu allow in "Apache"
	   
	   apt install -y php libapache2-mod-php php-mysql
	   rm /var/www/html/index.html
	   touch /var/www/html/index.php
	   echo -e "<?php \\$conn = new mysqli('#{db_ip}', 'root', '#{sql_pwd}');\n" >> /var/www/html/index.php
	   echo -e "if (\\$conn) { echo 'DB Connected'; } else { echo 'DB Not connected'; }\n" >> /var/www/html/index.php
	   echo -e "phpinfo(); ?>" >> /var/www/html/index.php
     SHELL

    end   #end of the web server do block
  
    #another block that defines the db server.
    #we'll call it simply "db"
    config.vm.define "db" do |db|
  
     db.vm.box = "ubuntu/focal64"
  
     #the VM will automatically get a NAT network connection
     #(outbound only) to get to the internet.
     #here connect the VM to the same private "host only" as
     #the web server
     db.vm.network "private_network", ip: db_ip

     #copy any needed files out to the vm

     #set up a provision block to install and configure the DB
     db.vm.provision "shell", inline: <<-SHELL
		apt update
		apt upgrade -y
		
		debconf-set-selections <<< "mysql-server mysql-server/root_password password #{sql_pwd}"
		debconf-set-selections <<< "mysql-server mysql-server/root_password_again password #{sql_pwd}"
		
		apt install -y mysql-server
     SHELL
  
    end  #endof the db block
  end
