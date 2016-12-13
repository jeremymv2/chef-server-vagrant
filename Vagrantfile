# encoding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :

CHEFSERVER_DEB = ENV['CHEFSERVER_DEB'] ||
  "$(curl -s https://downloads.chef.io/chef-server/stable | grep -o 'http[^\"]*ubuntu/14.04/chef-server[^\"]*amd64.deb' | head -1)"

AUTOMATESERVER_DEB = ENV['AUTOMATESERVER_DEB'] ||
  "$(curl -s https://downloads.chef.io/automate/stable | grep -o 'http[^\"]*ubuntu/14.04/delivery[^\"]*amd64.deb' | head -1)"

PUSHJOBSSERVER_DEB = ENV['PUSHJOBSSERVER_DEB'] ||
  "$(curl -s https://downloads.chef.io/push-jobs-server/stable | grep -o 'http[^\"]*ubuntu/14.04/opscode-push-jobs-server[^\"]*amd64.deb' | head -1)"

COMPLIANCE_DEB = ENV['COMPLIANCE_DEB'] ||
  "$(curl -s https://downloads.chef.io/compliance/stable | grep -o 'http[^\"]*ubuntu/14.04/chef-compliance[^\"]*amd64.deb' | head -1)"

COMPLIANCE_SCRIPT = <<EOF.freeze
deb=$(find /home/vagrant/pkg -name '*.deb' | tail -1)
echo "Installing Chef Compliance $deb"
sudo dpkg -i $deb
sudo chef-compliance-ctl reconfigure
sudo chef-compliance-ctl restart
EOF

COMPLIANCE_UPSTREAM_SCRIPT = <<EOF.freeze
echo "Install Chef Compliance"
apt-get update
apt-get install -y curl
PKG=#{COMPLIANCE_DEB}
BN="$(basename $PKG)"
test ! -f "/downloads/$BN" &&\
  echo "Downloading $PKG" &&\
  wget -qO "/downloads/$BN" "$PKG"

# add stuff to etc/hosts
echo "192.168.200.100 chef-server.test" | tee -a /etc/hosts
echo "192.168.200.103 automate-server.test" | tee -a /etc/hosts

dpkg -i "/downloads/$BN"
echo "verify_tls false" > /etc/chef-compliance/chef-compliance.rb
chef-compliance-ctl reconfigure --accept-license
chef-compliance-ctl restart

# copy environemt variables to share
mkdir -p /vagrant/env/compliance
cp /opt/chef-compliance/sv/core/env/* /vagrant/env/compliance

# copy config
mkdir -p /vagrant/config/compliance
cp /etc/chef-compliance/dex-connectors.json /vagrant/config/compliance
EOF

AUTOMATE_SERVER_SCRIPT = <<EOF.freeze
# pre-reqs
apt-get update
apt-get -y install curl

# ensure the time is uptodate
apt-get -y install ntp
service ntp stop
ntpdate -s time.nist.gov
service ntp start

PKG=#{AUTOMATESERVER_DEB}
BN="$(basename $PKG)"
test ! -f "/downloads/$BN" &&\
  echo "Downloading $PKG" &&\
  wget -qO "/downloads/$BN" "$PKG"

dpkg -i "/downloads/$BN"

mkdir -p /var/opt/delivery/license
cp -f /vagrant/delivery.license /var/opt/delivery/license
mkdir -p /etc/delivery
chmod 0644 /etc/delivery
cp -f /vagrant/delivery.pem /etc/delivery/delivery.pem
# cp -f /vagrant/delivery.rb /etc/delivery
# add other hosts to etc/hosts
echo "192.168.200.100 chef-server.test" | tee -a /etc/hosts
automate-ctl setup --license /var/opt/delivery/license/delivery.license --enterprise brewinc --no-build-node --key /etc/delivery/delivery.pem --server-url https://chef-server.test/organizations/brewinc --fqdn automate-server.test --no-configure
automate-ctl reconfigure
sleep 15
automate-ctl create-enterprise brewinc --ssh-pub-key-file /etc/delivery/builder_key.pub
automate-ctl reset-password brewinc admin insecurepassword
#ssh-keygen -t rsa -N '' -b 2048 -f /etc/delivery/builder_key
#echo "data_collector['token'] = '93a49a4f2482c64126f7b6015e6b0f30284287ee4054ff8807fb63d9cbd1c506'" | tee -a /etc/delivery/delivery.rb

EOF

BUILD_NODE_SCRIPT = <<EOF.freeze
# pre-reqs
apt-get update
apt-get -y install curl

# ensure the time is uptodate
apt-get -y install ntp
service ntp stop
ntpdate -s time.nist.gov
service ntp start

echo "192.168.200.100 chef-server.test" | tee -a /etc/hosts
echo "192.168.200.103 automate-server.test" | tee -a /etc/hosts
EOF

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

#PKG=#{PUSHJOBSSERVER_DEB}
#BN="$(basename $PKG)"
#test ! -f "/downloads/$BN" &&\
#  echo "Downloading $PKG" &&\
#  wget -qO "/downloads/$BN" "$PKG"

#chef-server-ctl install opscode-push-jobs-server --path=/downloads/$BN
#opscode-push-jobs-server-ctl reconfigure

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

# create delivery org and user
chef-server-ctl user-create delivery Delivery User delivery@example.com insecurepassword --filename delivery.pem
chef-server-ctl org-user-add brewinc delivery --admin


echo "Sync admin and validator keys"
cp -f /etc/opscode/webui_priv.pem /vagrant
cp -f /etc/opscode/pivotal.pem /vagrant
cp -f /home/vagrant/adminkey.pem /vagrant
cp -f /home/vagrant/delivery.pem /vagrant
cp -f /home/vagrant/brewinc-validator.pem /vagrant

# add hosts to /etc/hosts
echo "192.168.200.103 automate-server.test" | tee -a /etc/hosts
echo "192.168.200.101 automate-build-node.test" | tee -a /etc/hosts
echo "192.168.200.104 compliance-server.test" | tee -a /etc/hosts

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
echo "192.168.200.100 chef-server.test" | tee -a /etc/hosts
echo "192.168.200.103 automate-server.test" | tee -a /etc/hosts
echo "192.168.200.104 compliance-server.test" | tee -a /etc/hosts
EOF

def set_hostname(server)
  server.vm.provision 'shell', inline: "hostname #{server.vm.hostname}"
end

Vagrant.configure(2) do |config|

  config.vm.define 'chef-server' do |server|
    server.vm.box = 'bento/ubuntu-14.04'
    server.vm.box_url = 'http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_ubuntu-14.04_chef-provisionerless.box'
    server.vm.hostname = 'chef-server.test'
    server.vm.network 'private_network', ip: '192.168.200.100'
    server.vm.synced_folder './test-chef-server/', '/vagrant'
    server.vm.synced_folder './test-chef-server/.cache', '/downloads'
    server.vm.provision 'shell', inline: CHEF_SERVER_SCRIPT.dup
    set_hostname(server)

    server.vm.provider 'virtualbox' do |v|
      v.memory = 2048
      v.cpus = 1
    end
  end

  config.vm.define 'automate-build-node' do |server|
    server.vm.box = 'bento/ubuntu-14.04'
    server.vm.box_url = 'http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_ubuntu-14.04_chef-provisionerless.box'
    server.vm.hostname = 'automate-build-node.test'
    server.vm.network 'private_network', ip: '192.168.200.101'
    server.vm.synced_folder './test-chef-server/', '/vagrant'
    server.vm.synced_folder './test-chef-server/.cache', '/downloads'
    server.vm.provision 'shell', inline: BUILD_NODE_SCRIPT.dup
    set_hostname(server)

    server.vm.provider 'virtualbox' do |v|
      v.memory = 2048
      v.cpus = 1
    end
  end

  config.vm.define 'automate-server' do |server|
    server.vm.box = 'bento/ubuntu-14.04'
    server.vm.box_url = 'http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_ubuntu-14.04_chef-provisionerless.box'
    server.vm.hostname = 'automate-server.test'
    server.vm.network 'private_network', ip: '192.168.200.103'
    server.vm.synced_folder './test-chef-server/', '/vagrant'
    server.vm.synced_folder './test-chef-server/.cache', '/downloads'
    server.vm.provision 'shell', inline: AUTOMATE_SERVER_SCRIPT.dup
    set_hostname(server)

    server.vm.provider 'virtualbox' do |v|
      v.memory = 2048
      v.cpus = 1
    end
  end

  config.vm.define 'compliance-server' do |server|
    server.vm.box = 'bento/ubuntu-14.04'
    server.vm.box_url = 'http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_ubuntu-14.04_chef-provisionerless.box'
    server.vm.hostname = 'compliance-server.test'
    server.vm.network 'private_network', ip: '192.168.200.104'

    server.vm.provision 'shell', inline: COMPLIANCE_UPSTREAM_SCRIPT.dup
    set_hostname(server)

    server.vm.synced_folder './test-chef-server/', '/vagrant'
    File.exist?('./.cache') || Dir.mkdir('.cache')
    server.vm.synced_folder './.cache', '/downloads'

    server.vm.provider 'virtualbox' do |v|
      v.memory = 2048
      v.cpus = 1
    end
  end

  config.vm.define 'chef-client-node' do |server|
    server.vm.box = 'bento/ubuntu-14.04'
    server.vm.box_url = 'http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_ubuntu-14.04_chef-provisionerless.box'
    server.vm.hostname = 'node'
    server.vm.network 'private_network', ip: '192.168.200.102'
    server.vm.synced_folder './test-chef-server/', '/vagrant'
    config.vm.provision :shell, inline: NODE_SCRIPT.dup
    set_hostname(server)

    config.vm.provision :chef_client do |chef|
      # chef.version = '12.1.0'
      chef.chef_server_url = 'https://chef-server.test/organizations/brewinc'
      chef.validation_key_path = 'test-chef-server/brewinc-validator.pem' # local, will be uploaded
      chef.validation_client_name = 'brewinc-validator'
      chef.log_level = :info
      # chef.custom_config_path = "./test-chef-server/client.rb"

      chef.add_recipe 'audit_wrapper::default'
      # chef.json = { 'audit' => { 'collector' => 'chef-compliance', 'profiles' => { 'base/linux' =>  true } } }
    end
  end
end
