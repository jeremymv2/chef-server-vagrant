# encoding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :

CHEFSERVER_DEB = ENV['CHEFSERVER_DEB'] ||
                   "$(curl -s https://downloads.chef.io/chef-server/ubuntu/ | grep availableVersions | grep -o 'http[^\"]*ubuntu/14.04/chef-server[^\"]*amd64.deb' | head -1)"

CHEF_SERVER_SCRIPT = <<EOF.freeze
# install chef server
echo "Install Chef Server"
apt-get update
apt-get -y install curl

# ensure the time is uptodate
apt-get -y install ntp
service ntp stop
ntpdate -s time.nist.gov
service ntp start

PKG=#{CHEFSERVER_DEB}
BN="$(basename $PKG)"
test ! -f "/downloads/$BN" &&\
  echo "Downloading $PKG" &&\
  wget -qO "/downloads/$BN" "$PKG"

dpkg -i "/downloads/$BN"

chef-server-ctl reconfigure

# install chef manage
echo "Install Chef Manage"
chef-server-ctl install chef-manage
chef-server-ctl reconfigure
chef-manage-ctl reconfigure --accept-license

# restart services
chef-server-ctl restart

# create admin user
echo "Create admin and organization"
chef-server-ctl user-create admin Bob Admin admin@example.com insecurepassword --filename adminkey.pem
chef-server-ctl org-create brewinc "Brew, Inc." --association_user admin --filename brewinc-validator.pem

# create second demo user
chef-server-ctl user-create john John Doe john@example.com insecurepassword --filename johndoe.pem
chef-server-ctl org-create acmeinc "Acme, Inc." --association_user john --filename acme-validator.pem

echo "Sync admin and validator keys"
cp -f /etc/opscode/webui_priv.pem /vagrant
cp -f /etc/opscode/pivotal.pem /vagrant
cp -f /home/vagrant/adminkey.pem /vagrant
cp -f /home/vagrant/brewinc-validator.pem /vagrant

echo "Done with chef server initialization"
EOF

# make chef-server known by hostname
NODE_SCRIPT = <<EOF.freeze
echo "Prepare Chef Client node"
apt-get update
# ensure the time is uptodate
apt-get -y install ntp
service ntp stop
ntpdate -s time.nist.gov
service ntp start

# add other hosts to etc/hosts
echo "192.168.200.100 chef.test" | tee -a /etc/hosts
EOF

def set_hostname(server)
  server.vm.provision 'shell', inline: "hostname #{server.vm.hostname}"
end

Vagrant.configure(2) do |config|

  config.vm.define 'chef-server' do |server|
    server.vm.box = 'bento/ubuntu-14.04'
    server.vm.box_url = 'http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_ubuntu-14.04_chef-provisionerless.box'
    server.vm.hostname = 'chef.test'
    server.vm.network 'private_network', ip: '192.168.200.100'
    server.vm.synced_folder './test-chef-server/', '/vagrant'
    server.vm.synced_folder './test-chef-server/.cache', '/downloads'
    server.vm.provision 'shell', inline: CHEF_SERVER_SCRIPT.dup
    set_hostname(server)

    server.vm.provider 'virtualbox' do |v|
      v.memory = 2048
      v.cpus = 2
    end
  end

  config.vm.define 'chef-client-node' do |server|
    server.vm.box = 'bento/ubuntu-14.04'
    server.vm.box_url = 'http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_ubuntu-14.04_chef-provisionerless.box'
    server.vm.hostname = 'node'
    server.vm.network 'private_network', ip: '192.168.200.101'
    server.vm.synced_folder './test-chef-server/', '/vagrant'
    config.vm.provision :shell, inline: NODE_SCRIPT.dup
    set_hostname(server)

    config.vm.provision :chef_client do |chef|
      config.omnibus.chef_version = '12.0.1'
      chef.chef_server_url = 'https://chef.test/organizations/brewinc'
      chef.validation_key_path = 'test-chef-server/brewinc-validator.pem' # local, will be uploaded
      chef.validation_client_name = 'brewinc-validator'

      chef.log_level = :info

      #chef.add_recipe 'some_cookbook::default'
      #chef.json = { } # your json attributes
    end
  end
end
