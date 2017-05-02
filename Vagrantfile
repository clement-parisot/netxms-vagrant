	# -*- mode: ruby -*-
	# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "centos/7"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network "forwarded_port", guest: 8080, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    #vb.gui = true
 
    # Customize the amount of memory on the VM:
    vb.memory = "1024"
  end
 
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    yum update
    yum install -y gcc gcc-c++
    yum install -y postgresql-devel postgresql postgresql-server openssl-devel
    yum install -y wget
    cd /vagrant/
    if [ ! -f /vagrant/netxms-2.0.8.tar.gz ]; then
      wget https://www.netxms.org/download/netxms-2.0.8.tar.gz -O /vagrant/netxms-2.0.8.tar.gz
    fi
    tar xvfz netxms-2.0.8.tar.gz
    cd netxms-2.0.8
    mkdir -p /opt/netxms/
    sh ./configure --with-server --with-pgsql --with-agent
    make
    make install
    cp /vagrant/netxmsd.conf /etc/netxmsd.conf
    cp /vagrant/nxagentd.conf /etc/nxagentd.conf
    postgresql-setup initdb
    sed -i "s/ident/md5/g" /var/lib/pgsql/data/pg_hba.conf
    systemctl enable postgresql.service
    systemctl start postgresql.service
    sudo -u postgres psql -d template1 -c "CREATE USER netxms WITH PASSWORD 'ivaiL4ag';"
    sudo -u postgres createdb netxms_db
    sudo -u postgres psql -d template1 -c "GRANT ALL ON DATABASE netxms_db TO netxms;"
    /usr/local/bin/nxdbmgr init /usr/local/share/netxms/sql/dbinit_pgsql.sql 
    /usr/local/bin/nxagentd -d
    /usr/local/bin/netxmsd -d
    if [ ! -f /vagrant/nxmc-2.0.8.war ]; then
      wget https://www.netxms.org/download/webui/nxmc-2.0.8.war -O /vagrant/nxmc-2.0.8.war
    fi
    yum install -y tomcat tomcat-webapps tomcat-admin-webapps
    cp /vagrant/tomcat-users.xml /usr/share/tomcat/conf/tomcat-users.xml
    cp /vagrant/nxmc-2.0.8.war /usr/share/tomcat/webapps/nxmc.war
    systemctl start tomcat
    systemctl enable tomcat
  SHELL
end
