N_NODES = 5
PUBLIC_SUBNET = "192.168.11"
CLUSTER_SUBNET = "192.168.10"
ssh_pub_key = File.readlines("ssh/id_rsa_vagrant.pub").first.strip

Vagrant.configure("2") do |config|

  config.vm.box = "generic/ubuntu2004"

  config.vbguest.auto_update = false

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true # put true if you use your host to run ansible
  config.hostmanager.manage_guest = false # put true if you use vm to run ansible
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  config.vm.provider "virtualbox" do |vb|
    # Customize the amount of memory on the VM:
    vb.customize ["modifyvm", :id, "--groups", "/ceph"]
    vb.memory = 4096
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
          disk_file = File.absolute_path(".vagrant/machines/#{name}/virtualbox/disk-#{d}.vdi")
          unless File.exist?(disk_file)
            vb.customize ['createhd', '--filename', disk_file, '--size', '10240']
            if d == 1
              vb.customize ['storagectl', :id, '--name', 'OSD Controller', '--add', 'SCSI']
            end
          end
          vb.customize [
                         'storageattach', :id,
                         '--storagectl', 'OSD Controller',
                         '--port', d - 1,
                         '--device', 0,
                         '--type', 'hdd',
                         '--medium', disk_file
                       ]
        end
      end
      node.vm.provision "shell" do |shell|
        shell.inline = <<-SHELL
          if ! grep -q "#{ssh_pub_key}" "/home/vagrant/.ssh/authorized_keys"; then
            echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
          fi
        SHELL
      end
    end
  end

  # uncomment if you want to use vm to run ansible
  # config.vm.define "ansible" do |ansible|
  #
  #   ip = "#{PUBLIC_SUBNET}.1"
  #   ansible.vm.hostname = "ansible"
  #   ansible.vm.network :private_network, :ip => ip
  #   ansible.vm.provider :virtualbox do |vb|
  #     vb.name = "ansible"
  #   end
  #   # disable ipv6
  #   ansible.vm.provision "shell", inline: "sysctl -w net.ipv6.conf.all.disable_ipv6=1"
  #
  #   # add ssh keys ans config
  #   ansible.vm.provision "shell" do |shell|
  #     ssh_priv_key = File.read("ssh/id_rsa_vagrant").strip
  #     config = File.read("ssh/config").strip
  #     ansible_nodes = File.read("ssh/config.d/ansible_nodes").strip
  #     shell.inline = <<-SHELL
  # mkdir -p /home/vagrant/.ssh/config.d
  # cat <<EOL > /home/vagrant/.ssh/id_rsa_vagrant.pub
  # #{ssh_pub_key}
  # EOL
  #
  # cat <<EOL > /home/vagrant/.ssh/id_rsa_vagrant
  # #{ssh_priv_key}
  # EOL
  #
  # chmod 600 /home/vagrant/.ssh/id_rsa_vagrant.pub
  # chmod 600 /home/vagrant/.ssh/id_rsa_vagrant
  # chown vagrant: /home/vagrant/.ssh/id_rsa_vagrant.pub
  # chown vagrant: /home/vagrant/.ssh/id_rsa_vagrant
  #
  # cat <<EOL > /home/vagrant/.ssh/config
  # #{config}
  # EOL
  #
  # cat <<EOL > /home/vagrant/.ssh/config.d/ansible_nodes
  # #{ansible_nodes}
  # EOL
  #       SHELL
  #   end
  #   # install pip
  #   ansible.vm.provision "shell" do |shell|
  #     # install apt install python3-pip
  #     shell.inline = <<-SHELL
  #         add-apt-repository -y ppa:ansible/ansible
  #         apt update
  #         apt install -y python3-pip
  #       SHELL
  #   end
  # end
end
