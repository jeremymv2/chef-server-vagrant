# chef-server-vagrant
Test chef-server via vagrant / virtualbox

# Prerequisites
 * Vagrant
 * Virtualbox
 * put your Automate license into `test-chef-server/delivery.license`
 * add entries into your `hosts` file:

```
192.168.200.100 chef-server.test
192.168.200.101 automate-build-node.test
192.168.200.102 clientnode
192.168.200.103 automate-server.test
192.168.200.104 compliance-server.test
```

# Usage
 * vagrant status
 * vagrant up chef-server # bring up chef-server first
 * vagrant up chef-client-node
 * vagrant up ..
