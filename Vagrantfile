# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.include_offline = false
  config.hostmanager.ignore_private_ip = false

  servers = (1..3).map { |idx| "nomad.servers#{idx}" }
  clients = ["nomad.clients1"]
  groups = {
    "consul_servers" => servers,
    "consul_clients" => clients,
    "nomad_servers" => servers,
    "nomad_clients" => clients,
    "nomad:children" => [
      "nomad_servers",
      "nomad_clients"
    ]
  }

  host_groups = groups.reject { |k, v| k.end_with?(":children") || k.end_with?(":vars") }
  machines = host_groups.values.reduce([]) { |a, b| a | b }
  machines.each_with_index do |name, idx|
    config.vm.define name do |instance|
      instance.vm.network :private_network, ip: "192.168.50.#{2 + idx}"
      instance.vm.hostname = name

      instance.vm.provision "shell" do |s|
        s.inline = "sudo apt-get install -y python"
      end

      if name == machines.last
        instance.vm.provision "ansible" do |ansible|
          ansible.groups = groups
          ansible.limit = "all"
          ansible.playbook = "test.yml"
          ansible.verbose = "vv"
          ansible.sudo = true
        end
      end
    end
  end
end
