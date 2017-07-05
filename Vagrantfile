# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # Provide necessary customization.
  # See https://forums.freebsd.org/threads/52717/
  config.vm.provider "virtualbox" do |vb|
    vb.memory = 1024
    vb.cpus = 1
    vb.customize ["modifyvm", :id, "--hwvirtex", "on"]
    vb.customize ["modifyvm", :id, "--audio", "none"]
    vb.customize ["modifyvm", :id, "--nictype1", "virtio"]
    vb.customize ["modifyvm", :id, "--nictype2", "virtio"]
  end

  # Use FreeBSD 11.0 RELEASE-p1 since FreeBSD-11.0-RELEASE has been revoked.
  config.vm.box = "freebsd/FreeBSD-11.0-RELEASE-p1"
  config.vm.base_mac = "080027D14C66"

  # The default shell for the 'vagrant' user on this box is /bin/sh.
  config.ssh.shell = "/bin/sh"

  # BSD-based guests do not support the VirtualBox filesystem at this time.
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Bootstrapping Salt will initially fail in FreeBSD because of CA issues.
  # See https://github.com/mitchellh/vagrant/issues/7392
  config.vm.provision "shell", inline: <<-SHELL
    freebsd-update fetch install --not-running-from-cron
    pkg update
    pkg upgrade -y
    pkg install -y ca_root_nss
  SHELL

  # Master-specific configuration.
  config.vm.define :master, primary: true do |master_config|
    master_config.vm.host_name = "saltmaster.local"
    master_config.vm.network "private_network", ip: "192.168.50.10"

    # Share folders over rsync.
    master_config.vm.synced_folder "saltstack/salt/", "/usr/local/etc/salt/states", type: "rsync"
    master_config.vm.synced_folder "saltstack/pillar/", "/usr/local/etc/salt/pillar", type: "rsync"

     master_config.vm.provision :salt do |salt|
       salt.master_config = "saltstack/etc/master"
       salt.master_key = "saltstack/keys/master_minion.pem"
       salt.master_pub = "saltstack/keys/master_minion.pub"
	   salt.seed_master = {
                           "minion1" => "saltstack/keys/minion1.pub",
                           "minion2" => "saltstack/keys/minion2.pub"
                          }

       salt.install_type = "stable"
       salt.install_master = true
       salt.no_minion = true
       salt.verbose = true
       salt.colorize = true
       salt.bootstrap_options = "-c /tmp"
     end
  end

  # Configure our minions.
  (1..2).each do |i|
    config.vm.define "minion#{i}" do |minion_config|
      minion_config.vm.host_name = "saltminion#{i}.local"
      minion_config.vm.network "private_network", ip: "192.168.50.1#{i}"

      minion_config.vm.provision :salt do |salt|
        salt.minion_config = "saltstack/etc/minion#{i}"
        salt.minion_key = "saltstack/keys/minion#{i}.pem"
        salt.minion_pub = "saltstack/keys/minion#{i}.pub"
        salt.install_type = "stable"
        salt.verbose = true
        salt.colorize = true
        salt.bootstrap_options = "-c /tmp"
      end
    end
  end
end
