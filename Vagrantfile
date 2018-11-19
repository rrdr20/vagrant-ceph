DOMAIN_NAME = "dw002.local"

# Defines the specifications for the nodes being deployed.
nodes = [
    # Deploy a server for running services such as ansible for configuration deployment.
    {:base_name => "srv", 
     :num_vms => "1", 
     :box => "ubuntu/bionic64", 
     :cpu => "1",
     :ram => "512",
     :networks => [{:name => "ceph_net01", :base_ip => "192.168.20.10"}]},
    
    # Deploy Monitor servers for the Ceph Cluster.
    {:base_name => "mon", 
     :num_vms => "3", 
     :box => "ubuntu/bionic64", 
     :cpu => "1",
     :ram => "512",
     :networks => [{:name => "ceph_net01", :base_ip => "192.168.20.20"}]},
    
    # Deploy OSD servers for the Ceph cluster.
    {:base_name => "osd", 
     :num_vms => "5", 
     :box => "ubuntu/bionic64", 
     :cpu => "1",
     :ram => "1024",
     :disks => {:num => "3", :size => "20480"},
     :networks => [{:name => "ceph_net01", :base_ip => "192.168.20.30"},
                   {:name => "ceph_net02", :base_ip => "192.168.21.30"}]}
]

Vagrant.configure("2") do |config|
    nodes.each do |node|
        (1..node[:num_vms].to_i).each do |srv_count|
            node_name = node[:base_name] + "0#{srv_count}.#{DOMAIN_NAME}"

	    config.vm.define node_name do |nodecfg|
	        nodecfg.vm.box = node[:box]
		nodecfg.vm.hostname = node_name
		nodecfg.vm.synced_folder ".", "/vagrant", disabled: true

                # Some of the servers will have more than one network interface defined
                node[:networks].each do |nodeint|
		    int_ip = nodeint[:base_ip][0..-2] + "#{srv_count}"
                    nodecfg.vm.network "private_network", ip: int_ip,
		                                          nic_type: "82540EM",
							  virtualbox__intnet: nodeint[:name]
                end

                nodecfg.vm.provider "virtualbox" do |vb|
                    vb.cpus = node[:cpu]
                    vb.memory = node[:ram]

                    # Disable the audio controller, not needed
                    vb.customize ["modifyvm", :id, "--audio", "none"]
                  
                    # Add disks to the node if defined
                    if node.has_key?(:disks)
                        (2..node[:disks][:num].to_i + 1).each do |dskid|
                            # Check if disk is already created if not VB will throw error
                            unless File.exist?("#{node_name}-disk0#{dskid}.vdi")
                                vb.customize ["createmedium", "disk",
				              "--filename", "#{node_name}-disk0#{dskid}.vdi",
                                              "--size", "#{node[:disks][:size]}"]
                            end
                            # Begin adding at the 3rd position on the controller as image and config drives are already attached.
                            vb.customize ["storageattach", :id,
			                  "--storagectl", "SCSI",
					  "--device", "0",
					  "--port", "#{dskid}",
                                          "--type", "hdd",
					  "--medium", "#{node_name}-disk0#{dskid}.vdi"]
                        end
                    end
                end

                nodecfg.ssh.insert_key = false
                nodecfg.ssh.private_key_path = ["keys/id_rsa", "~/.vagrant.d/insecure_private_key"]
                nodecfg.vm.provision :file,
                    source: "keys/id_rsa",
                    destination: "/home/vagrant/.ssh/id_rsa"
                nodecfg.vm.provision :file,
                    source: "keys/id_rsa.pub",
                    destination: "/home/vagrant/.ssh/id_rsa.pub"
                nodecfg.vm.provision :file,
                    source: "keys/id_rsa.pub",
                    destination: "/home/vagrant/.ssh/authorized_keys"
                nodecfg.vm.provision :shell, inline: <<-EOC
                    chmod 600 /home/vagrant/.ssh/id_rsa
                    chmod 644 /home/vagrant/.ssh/id_rsa.pub
                EOC
            end
        end
    end
 end

