# -*- mode: ruby -*-
# vi: set ft=ruby :

# Check that our required plugins are installed.
require './lib/plugins/plugins'

Vagrant.configure(2) do |config|
  # Since we change the SSH user, we need to first install a public key. This
  # is done on the initial provisioning, which needs to run as the 'vagrant'
  # user. So, we switch based on the presence of a semaphore file, which we
  # create on provisioning, and remove after destroy.
  project_root = File.expand_path(File.dirname(__FILE__))
  first_run = !File.file?("#{project_root}/.first_run_complete")
  config.trigger.after [:destroy] do
    system("rm .first_run_complete > /dev/null 2>&1; echo '==> Removing .first_run_complete'")
  end

  hostname = "aegir3.local"

  # Silence annoying and spurious warnings
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  config.ssh.forward_agent = true

  config.vm.define "default" do |vm1|

    vm1.vm.hostname = hostname

    vm1.vm.provider "virtualbox" do |vbox|
      vbox.memory = "1024"
    end

    #vm1.vm.box = "ubuntu/trusty64"
    vm1.vm.box = "hashicorp/precise64"

    vm1.vm.network "private_network", ip: "10.42.0.10"

    if first_run
      vm1.vm.provision "file",
        source: "~/.ssh/id_rsa.pub",
        destination: "/vagrant/authorized_keys"
    else
      # Mount platforms via SSHFS
      config.sshfs.paths = { "/var/aegir/platforms" => "./platforms" }
      config.sshfs.enabled = false
      config.sshfs.username = "aegir"
      # SSH as the 'aegir' user
      config.ssh.username = 'aegir'
      config.ssh.private_key_path = '~/.ssh/id_rsa'
      # Copy in some user-specific files to make the environment more familiar
      dot_files = ['.gitconfig', '.vimrc', '.bashrc']
      dot_files.each do |dot_file|
        vm1.vm.provision "file", source: "~/#{dot_file}",
          destination: "/var/aegir/#{dot_file}"
      end
    end

    vm1.vm.provision "puppet",
      module_path: "modules",
      facter: { "fqdn" => hostname },
      run: "always"

  end

end
