N_NODES = 5
PUBLIC_SUBNET = "192.168.11"
CLUSTER_SUBNET = "192.168.10"
ssh_pub_key = File.readlines("ssh/id_rsa_vagrant.pub").first.strip

Vagrant.configure("2") do |config|

  config.vm.box = "generic/ubuntu2004"

  config.vbguest.auto_update = false

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  config.vm.provider "virtualbox" do |vb|
    # Customize the amount of memory on the VM:
    vb.customize ["modifyvm", :id, "--groups", "/ceph"]
    vb.memory = 2048
    vb.cpus = 4
  end

  (1..N_NODES).each do |i|
    name = "node-#{i}"
    ip = "#{PUBLIC_SUBNET}.1#{ i}"
    config.vm.define name do |node|
      node.vm.hostname = name
      node.vm.network :private_network, :ip => ip
      node.vm.network :private_network, :ip => "#{CLUSTER_SUBNET}.1#{i}"
      # Virtualbox
      node.vm.provider :virtualbox do |vb|
        vb.name = name
        (1..2).each do |d|
          disk_file = "D:/vbox/vms/ceph/#{name}/disk-#{d}.vdi"
          unless File.exist?(disk_file)
            vb.customize ['createhd', '--filename', disk_file, '--size', '10240']
          end
          vb.customize [
                         'storageattach', :id,
                         '--storagectl', 'SCSI',
                         '--port', 1 + d,
                         '--device', 0,
                         '--type', 'hdd',
                         '--medium', disk_file
                       ]
        end
      end
      node.vm.provision "shell" do |shell|
        shell.inline = <<-SHELL
          echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
          echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
        SHELL
      end
    end
  end

  config.vm.define "ansible" do |ansible|

    ip = "#{PUBLIC_SUBNET}.1"
    ansible.vm.hostname = "ansible"
    ansible.vm.network :private_network, :ip => ip
    ansible.vm.provider :virtualbox do |vb|
      vb.name = "ansible"
    end

    ansible.vm.provision "shell" do |shell|
      ssh_priv_key = File.readlines("ssh/id_rsa_vagrant").first.strip
      config = File.readlines("ssh/config").first.strip
      ansible_nodes = File.readlines("ssh/config.d/ansible_nodes").first.strip
      shell.inline = <<-SHELL
        mkdir -p /home/vagrant/.ssh/config.d
        echo #{ssh_pub_key} >> /home/vagrant/.ssh/id_rsa_vagrant.pub
        echo #{ssh_priv_key} >> /home/vagrant/.ssh/id_rsa_vagrant
        chmod 600 /home/vagrant/.ssh/id_rsa_vagrant.pub
        chmod 600 /home/vagrant/.ssh/id_rsa_vagrant
        echo #{config} >> /home/vagrant/.ssh/config
        echo #{ansible_nodes} >> /home/vagrant/.ssh/config.d/ansible_nodes
      SHELL
      # install apt install python3-pip
      shell.inline = <<-SHELL
        apt update
        apt install -y python3-pip
      SHELL
    end
  end
end
