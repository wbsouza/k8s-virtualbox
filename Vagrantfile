IMAGE_NAME = "bento/ubuntu-18.04"
K8S_NAME = "k8s-cluster"
IP_BASE = "192.168.50."

MASTERS_QTY = 1
WORKERS_QTY = 3
MEM = 4096
CPU = 2


Vagrant.configure("2") do |config|
	
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
        v.memory = MEM
        v.cpus = CPU
    end

    (1..MASTERS_QTY).each do |i|      
        config.vm.define "master#{i}" do |master|
            master.vm.box = IMAGE_NAME
            master.vm.network "private_network", ip: "#{IP_BASE}#{i + 10}"
            master.vm.hostname = "master#{i}"
            master.vm.provision "ansible" do |ansible|
                ansible.playbook = "roles/k8s.yml"
                #Generate Ansible Groups for inventory
                ansible.groups = {
                    "k8s-master" => ["master[1:#{MASTERS_QTY}]"],
                    "k8s-node" =>   ["worker[1:#{WORKERS_QTY}]"]
                }
                #Redefine defaults
                ansible.extra_vars = {
                    k8s_cluster_name:       K8S_NAME,                    
                    k8s_master_admin_user:  "vagrant",
                    k8s_master_admin_group: "vagrant",
                    k8s_master_apiserver_advertise_address: "#{IP_BASE}#{i + 10}",
                    k8s_master_node_name: "master#{i}",
                    k8s_node_public_ip: "#{IP_BASE}#{i + 10}"
                }                
            end
        end
    end

    (1..WORKERS_QTY).each do |j|
        config.vm.define "worker#{j}" do |node|
            node.vm.box = IMAGE_NAME
            node.vm.network "private_network", ip: "#{IP_BASE}#{j + 10 + MASTERS_QTY}"
            node.vm.hostname = "worker#{j}"
            node.vm.provision "ansible" do |ansible|
                ansible.playbook = "roles/k8s.yml"
                #Generate Ansible Groups for inventory              
                ansible.groups = {
                    "k8s-master" => ["master[1:#{MASTERS_QTY}]"],
                    "k8s-node" =>   ["worker[1:#{WORKERS_QTY}]"]
                }                    
                #Redefine defaults
                ansible.extra_vars = {
                    k8s_cluster_name:     K8S_NAME,
                    k8s_node_admin_user:  "vagrant",
                    k8s_node_admin_group: "vagrant",
                    k8s_node_public_ip: "#{IP_BASE}#{j + 10 + MASTERS_QTY}"
                }
            end
        end
    end
end    

