# vi: set ft=ruby :

# Builds multiple VMs using a JSON config file
# Author: Ernesto Iser

# read vm and chef configurations from JSON file
nodes_config = (JSON.parse(File.read("nodes.json")))['nodes']

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "centos/7"

  nodes_config.each do |node|
    node_name   = node[0] # name of node
    node_values = node[1] # content of node

    config.vm.define node_name do |config|
      # configures all forwarding ports in JSON array
      ports = node_values['ports']
      ports.each do |port|
        config.vm.network :forwarded_port,
          host:  port[':host'],
          guest: port[':guest'],
          id:    port[':id']
      end

      config.vm.hostname = node_name
      config.vm.network :private_network, ip: node_values[':ip']
      config.vbguest.auto_update = false

      config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", node_values[':memory']]
        vb.customize ["modifyvm", :id, "--name", node_name]
      end

      config.vm.provision "chef_zero" do |chef|
        role = node_values[':role']

        # Specify the local paths where Chef data is stored
        chef.cookbooks_path = "provisioning/chef/cookbooks"
        chef.roles_path = "provisioning/chef/roles"
        chef.data_bags_path = "provisioning/chef/data_bags"

        chef.add_role role
      end
    end
  end
end
