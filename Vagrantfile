require 'yaml'

debug = false

current_dir = File.dirname(__FILE__)

# Load all configuration from a single yaml file
conf = YAML.load_file("#{current_dir}/config/config.yml")

if debug
  puts "Galaxy configuration parameters:"
  puts conf['galaxy']
  puts "VM configuration parameters:"
  puts conf['vm']
end 

def get_credentials()
  if ENV['GALAXY_USER'].nil? or ENV['GALAXY_PASSWORD'].nil? then
    puts 'Using username and password from ./config/config.yml to register the user in Galaxy.'
    puts 'To override this behavior, please add the following in your ~/.bashrc file:'
    puts '  export GALAXY_USER="AAFC-UID@agr.gc.ca"'
    puts '  export GALAXY_PASSWORD="<A TEMP PASSWORD>"'
  else
    puts 'Using GALAXY_USER=%s parameter from environment ' % ENV['GALAXY_USER']
    conf['galaxy']['user'] = ENV['GALAXY_USER']
    conf['galaxy']['password'] = ENV['GALAXY_PASSWORD']
  end
end

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.
  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = conf['vm']['box']

  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  config.vm.box_url = conf['vm']['box_url']

  config.vm.provider "virtualbox" do |v|
    v.gui = true
    v.customize ["modifyvm", :id, '--cpus', conf['vm']['cpus']]
    v.customize ["modifyvm", :id, "--memory", conf['vm']['memory']]
  end

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  conf['vm']['port_forward'].each do |guest_port, host_port|
    config.vm.network :forwarded_port, guest: guest_port, host: host_port
  end

#  config.vm.provider "parallels" do |pa, override|
#  end

#  config.vm.provider "virtualbox" do |vb, override|
#  end


  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network :private_network, ip: conf['vm']['ip']

  # Install puppet modules rather than installing from puppet source
  # Needed to extend/tweak modules
  config.vm.provision "shell", path: "install_puppet_modules.sh"

  # Provision puppet based on installed modules
  config.vm.provision "puppet" do |puppet|
    puppet.manifests_path = "manifests/"
    puppet.manifest_file = "default.pp"
    # note that this appears to be the path on the host, not within the guest
    puppet.hiera_config_path = "hiera.yaml"
    puppet.options = "--verbose --debug"
  end
  
end
