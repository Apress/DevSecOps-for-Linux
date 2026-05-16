# -*- mode: ruby -*-
# vi: set ft=ruby :

required_plugins = ["vagrant-disksize", "vagrant-reload", "vagrant-scp"]

def plugin_installed?(plugin)
  installed = `vagrant plugin list`
  installed.include?(plugin)
end

required_plugins.each do |plugin|
  unless plugin_installed?(plugin)
    puts "Plugin '#{plugin}' is not installed. Installing now..."
    system("vagrant plugin install #{plugin}")
    puts "Plugin '#{plugin}' installed successfully."
    puts "Please reload or re-run your Vagrant command after the plugin installation completes."
    exit 1
  end
end

Vagrant.configure("2") do |config|

  case RUBY_PLATFORM
  when /x86_64|amd64|x64-mingw/
    config.vm.box = "oraclelinux/9"
    config.vm.hostname = "ol9-x86_64"
    config.vm.box_version = "9.5.661"
  when /aarch64|arm64/
    config.vm.box = "oraclelinux/9-aarch64"
    config.vm.hostname = "ol9-arm"
    config.vm.box_version = "9.5.129"
  else
    raise "Unsupported architecture: #{RUBY_PLATFORM}"
  end
  host_fqdn = "lab01.carcano.corp"
  vm_memory = 16384
  vm_cpus = 6
  primary_disk_size = '100GB'
  config.vm.define "devsecops_lab_01" do |node|
    node.vm.box = config.vm.box
    node.vm.hostname = host_fqdn
    node.vm.network "public_network", bridge: "en0: Ethernet", type: "dhcp"
    node.vm.provider "virtualbox" do |vb|
      vb.name = "devsecops_lab_01"
      vb.memory = vm_memory
      vb.cpus = vm_cpus
    end
    node.disksize.size = primary_disk_size 
  end
  config.vm.provision "shell", inline: <<-SHELL
    sudo dnf install -y cloud-utils-growpart
    sudo growpart /dev/sda 3
    sudo lvresize -l100%VG vg_main/lv_root
    sudo xfs_growfs /
    sudo sed -i '/PasswordAuthentication /d' /etc/ssh/sshd_config.d/90-vagrant.conf
    sudo systemctl restart sshd
  SHELL
  config.vm.synced_folder '.', '/vagrant', disabled: false
end

