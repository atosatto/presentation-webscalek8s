# -*- mode: ruby -*-
# vi: set ft=ruby :

# Number of Kubernetes Nodes
$num_masters = (ENV['NUM_MASTERS'] || 1).to_i
$num_minions = (ENV['NUM_MINIONS'] || 2).to_i

Vagrant.configure(2) do |config|

  # Extra varibles to pass to the Ansible provisioner.
  # Defined here to allow provider specific customizations.
  $extra_vars = Hash.new()

  def set_digital_ocean(provider, override)
    override.ssh.private_key_path = '~/.ssh/id_rsa'
    override.vm.box = 'digital_ocean'
    override.vm.box_url = "https://github.com/devopsgroup-io/vagrant-digitalocean/raw/master/box/digital_ocean.box"

    provider.token = ENV['DIGITAL_OCEAN_TOKEN']
    provider.image = 'centos-7-0-x64'
    provider.region = 'lon1'
    provider.size = '1gb'
    provider.private_networking = true # enable private network interface
  end

  def set_vbox(vb, config)
    config.vm.box = "centos/7"
    config.vm.network "private_network", type: "dhcp"
    vb.gui = false
    vb.memory = 2048
    vb.cpus = 2

    # Use faster paravirtualized networking
    vb.customize ["modifyvm", :id, "--nictype1", "virtio"]
    vb.customize ["modifyvm", :id, "--nictype2", "virtio"]

    # Customizing the ansible provisioning
    $extra_vars = $extra_vars.merge({
      # We cluster etcd on the private_network iface (eth1)
      "etcd_peer_iface" => "ansible_eth1"
      # for some hypervisors, like vbox, eth0 is not the right interface to bind to.
      # ansible.extra_vars = { flannel_opts: "--iface=eth1"}
    })

  end

  def set_provider(n)
    n.vm.provider :digital_ocean do |os, override|
      set_digital_ocean(os, override)
    end
    n.vm.provider :virtualbox do |vb, override|
      set_vbox(vb, override)
    end
  end

  # Kubernetes Minions
  minions = Array.new()
  $num_minions.times do |i|
    name = "kube-node-#{i+1}"
    minions.push(name)
    config.vm.define "#{name}" do |n|
      n.vm.hostname = name
      set_provider(n)
    end
  end

  # Kubernetes Masters
  masters = Array.new()
  $num_masters.times do |i|
    name = "kube-master-#{i+1}"
    masters.push(name)
    config.vm.define "#{name}" do |n|
      n.vm.hostname = name
      set_provider(n)

      # Once created the last kube-master, we run provisioning
      if i == $num_masters - 1

        # Creates the ansible inventory, see it in .vagrant
        groups = {
           "masters" => masters,
           "etcd:children" => "masters",
           "nodes" => minions,
           "nfs-server:children" => "masters",
           "nfs-client:children" => "nodes",
           "nfs:children" => ["nfs-server", "nfs-client"],
           "all_groups:children" => ["nfs","etcd","masters","nodes"]
        }

        # Customizing the Kubernetes provisioning
        $extra_vars = $extra_vars.merge({
          # Enables HPA, deployments features.
          "apiserver_extra_args" => "--runtime-config=extensions/v1beta1=true"
        })

        # Creates /etc/hosts so machines can talk via their
        # 'internal' IPs instead of the public ip.
        n.vm.provision :ansible do |ansible|
          ansible.groups = groups
          ansible.playbook = "./ansible-populate-hostsfile.yml"
          ansible.limit = "all" #otherwise the metadata wont be there for ipv4?
          ansible.raw_ssh_args = ['-o ControlMaster=no']
        end

        # Provisions both flannel and kubernetes.
        n.vm.provision :ansible do |ansible|
          ansible.groups = groups
          ansible.playbook = "./kubernetes-contrib/ansible/cluster.yml"
          ansible.limit = "all" #otherwise the metadata wont be there for ipv4?
          ansible.extra_vars = $extra_vars
          ansible.raw_ssh_args = ['-o ControlMaster=no']
        end
      end

      # Setup the NFS server in the nfs-server group
      n.vm.provision :ansible do |ansible|
        ansible.groups = groups
        ansible.playbook = "./ansible-nfs-server.yml"
        ansible.limit = "all" #otherwise the metadata wont be there for ipv4?
        ansible.raw_ssh_args = ['-o ControlMaster=no']
      end

    end

  end

end
