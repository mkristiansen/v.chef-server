host_cache_path = File.expand_path("../.cache", __FILE__)
guest_cache_path = "/tmp/vagrant-cache"

# ensure the cache path exists
FileUtils.mkdir(host_cache_path) unless File.exist?(host_cache_path)

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    # Detects vagrant-omnibus plugin
    if Vagrant.has_plugin?('vagrant-omnibus')
      puts 'INFO:  Vagrant-omnibus plugin detected.'
      config.omnibus.chef_version = "11.12.2"
    else
     puts "WARNING: Vagrant-omnibus plugin not detected. Please install the plugin with\n       'vagrant plugin install vagrant-omnibus' from any other directory\n       before continuing."
    end

    config.vm.define :chef do |chef_config|
 
    config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--cpus", 2]
      vb.customize ["modifyvm", :id, "--memory", 1024]
      vb.gui = true
    end

    # Do not upgrade to a later Ubuntu version. Chef cookbook does not currently support past 12.04 
    # See https://github.com/opscode-cookbooks/chef-server#platform
    chef_config.vm.box = "precise64.box"
    chef_config.vm.box_url = "http://files.vagrantup.com/precise64.box"
    chef_config.vm.network :private_network, :ip => "10.1.2.10"
    chef_config.vm.synced_folder host_cache_path, guest_cache_path
    chef_config.vm.hostname = "chef"

    #chef_config.hostmanager.enabled = true
    #chef_config.hostmanager.manage_host = true
    #chef_config.hostmanager.ignore_private_ip = false
    #chef_config.hostmanager.aliases = %w(chef-server.local chef-server chef.local)
    #chef_config.hostsupdater.aliases = ["chef-server.local", "chef-server", "chef.local"]

    VAGRANT_JSON = JSON.parse(Pathname(__FILE__).dirname.join('nodes', 'vagrant.json').read)

    chef_config.vm.provision :chef_solo do |chef|
       chef.cookbooks_path = ["site-cookbooks", "cookbooks"]
       chef.roles_path = "roles"
       chef.data_bags_path = "data_bags"
       chef.provisioning_path = guest_cache_path

       chef.run_list = VAGRANT_JSON.delete('run_list')
       chef.json = VAGRANT_JSON
       # old way
       #VAGRANT_JSON['run_list'].each do |recipe|
       # chef.add_recipe(recipe)
       #end if VAGRANT_JSON['run_list']

       Dir.glob(Pathname(__FILE__).dirname.join('roles', '*.json')).each do |role|
        chef.add_role(Pathname.new(role).basename(".*").to_s)
       end
    end

    chef_config.vm.provision :shell, :inline => "sudo cp /etc/chef-server/*.pem /vagrant/pem"
  end
end
