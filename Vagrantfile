# Run before "vagrant up"
# vagrant plugin install vagrant-vbguest vagrant-hostmanager vagrant-cachier
#
# Run before "vagrant provision"
# ansible-galaxy install -r app/ansible/requirements.txt

require 'rbconfig'

IP_ADDRESS = "172.22.22.137"
PROJECT_NAME = "testproject"

Vagrant.configure(2) do |config|
    config.vm.box = "ubuntu/trusty64"

    # Configure the network interfaces
    config.vm.network :forwarded_port, guest: 80, host: 8083

    # Configure shared folders
    config.vm.synced_folder ".", "/var/www", type: "nfs"
    config.vm.synced_folder "app/ansible", "/vagrant/ansible", type: "nfs"

    # Configure VirtualBox environment
    config.vm.provider :virtualbox do |v|
        v.name = PROJECT_NAME + "_vm"
        v.cpus = 2
        v.memory = 2048
        v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        v.customize ["modifyvm", :id, "--ioapic", "on"]
    end

    # Use hostonly network with a static IP Address and enable
    # hostmanager so we can have a custom domain for the server
    # by modifying the host machines hosts file
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.vm.define PROJECT_NAME do |node|
        node.vm.hostname = PROJECT_NAME + ".dev"
        node.vm.network :private_network, ip: IP_ADDRESS
        node.hostmanager.aliases = ["www." + PROJECT_NAME + ".dev"]
    end
    config.vm.provision :hostmanager

    # Provision the box
    is_windows = (RbConfig::CONFIG['host_os'] =~ /mswin|mingw|cygwin/)
    if is_windows
      # Provisioning configuration for shell script.
      config.vm.provision "shell" do |sh|
        sh.path = "app/ansible/JJG-Ansible-Windows/windows.sh"
        sh.args = "ansible/site.yml"
      end
    else
      # Provisioning configuration for Ansible (for Mac/Linux hosts).
      config.vm.provision "ansible" do |ansible|
        ansible.sudo = true
        ansible.playbook = "app/ansible/site.yml"
        ansible.extra_vars = { ansible_ssh_user: 'vagrant' }
      end
    end

    # Configure Cachier plugin
    if Vagrant.has_plugin?("vagrant-cachier")
        config.cache.scope = :box
        config.cache.enable :apt
        config.cache.enable :gem
        config.cache.enable :npm
        config.cache.enable :bower
        config.cache.enable :composer
        config.cache.auto_detect = false
        config.cache.synced_folder_opts = {
          type: :nfs,
          mount_options: ['rw', 'vers=3', 'tcp', 'nolock']
        }
    end
end
