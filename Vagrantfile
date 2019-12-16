# vi: set ft=ruby :
#


cluster_count = 1
node_data_disk_count = 2
driveletters = ('a'..'z').to_a
disk_size = 20 #GB
cpus = 2
memory = 2048

node_count = cluster_count * 3
total_disks = node_count * node_data_disk_count
total_disk_usage = total_disks * disk_size
enable_glusterd2_rest_auth = false

Vagrant.configure(2) do |config|
    puts "Creating #{cluster_count} clusters."
    puts "Creating #{node_count} nodes."
    puts "Creating #{node_data_disk_count} data disks (#{disk_size}G) each."
    puts "Using #{total_disk_usage} GB!."

    (1..node_count).reverse_each do |num|
      config.vm.define "node-#{num}" do |node|
        vm_ip = "192.168.250.#{num+10}"

        node.vm.box = "centos/7"
        node.vm.synced_folder ".", "/vagrant", disabled: true
        node.vm.network "private_network",
            :ip => vm_ip,
            :libvirt__driver_queues => "#{cpus}"
        node.vm.post_up_message = "VM private ip: #{vm_ip}"
        node.vm.hostname = "gd2-node-#{num}"

        node.vm.provider "libvirt" do |lvt|
          lvt.memory = "#{memory}"
          lvt.cpus = "#{cpus}"
          lvt.nested = false
          lvt.cpu_mode = "host-passthrough"
          lvt.volume_cache = "writeback"
          lvt.graphics_type = "none"
          lvt.video_type = "vga"
          lvt.video_vram = 1024
          # lvt.usb_controller :model => "none"  # (requires vagrant-libvirt 0.44 which is not in Fedora yet)
          lvt.random :model => 'random'
          lvt.channel :type => 'unix', :target_name => 'org.qemu.guest_agent.0', :target_type => 'virtio'
          #disk_config
          disks = []
          (2..(node_data_disk_count+1)).each do |d|
            lvt.storage :file, :device => "vd#{driveletters[d]}", :size => "#{disk_size}G"
            disks.push "/dev/vd#{driveletters[d]}"
	  end

        end

        if num == 1
          node.vm.synced_folder "./dist", "/root/.local/share/cockpit/gluster-management", type: "rsync", create: true, rsync__args: ["--verbose", "--archive", "--delete", "-z"]
          node.vm.synced_folder "./ansible/", "/etc/ansible//", type: "rsync", create: true, rsync__args: ["--verbose","--archive"]
          node.vm.network "forwarded_port", guest: 9090, host: 9091
          node.vm.post_up_message << "You can now access Cockpit at http://localhost:9091 (login as 'admin' with password 'foobar')"
        end

        node.vm.provision "shell", inline: <<-SHELL
          set -u


          yum update -y --disableplugin=subscription-manager --downloaddir=/dev/shm
          yum install -y --disableplugin=subscription-manager --downloaddir=/dev/shm util-linux

          echo foobar | passwd --stdin root
          getent passwd admin >/dev/null || useradd -c Administrator -G wheel admin
          echo foobar | passwd --stdin admin

          usermod -a -G wheel vagrant



          yum install -y --disableplugin=subscription-manager --downloaddir=/dev/shm \
              storaged \
              storaged-lvm2 \
              yum-utils \
              cockpit \
              python-requests \
              python-jwt \
              epel-release
              # ovirt-hosted-engine-setup
          curl -o /etc/yum.repos.d/gd2-master.repo http://artifacts.ci.centos.org/gluster/gd2-nightly/gd2-master.repo
          curl -o /etc/yum.repos.d/sac-gluster-ansible-epel-7.repo https://copr.fedorainfracloud.org/coprs/sac/gluster-ansible/repo/epel-7/sac-gluster-ansible-epel-7.repo
          curl -o /etc/yum.repos.d/glusterfs-nightly-master.repo http://artifacts.ci.centos.org/gluster/nightly/master.repo
          yum install -y --disableplugin=subscription-manager --downloaddir=/dev/shm \
            glusterd2 \
            gluster-ansible python-gluster-mgmt-client \
            glusterfs-server glusterfs-fuse glusterfs-api

          ansible localhost -m lineinfile -a "path=/etc/glusterd2/glusterd2.toml state=present regexp='^[^#]*restauth\s*=\s*' line=restauth=#{enable_glusterd2_rest_auth}"
          ansible localhost -m lineinfile -a "path=/etc/glusterd2/glusterd2.toml state=present regexp='^[^#]*peeraddress\s*=\s*' line=peeraddress='#{vm_ip}:24008'"
          ansible localhost -m lineinfile -a "path=/etc/ssh/sshd_config state=present regexp='^[#]*PasswordAuthentication\s+yes' line='PasswordAuthentication yes'"
          systemctl restart sshd

          systemctl enable --now cockpit.socket
          systemctl enable --now glusterd2
          printf "[WebService]\nAllowUnencrypted=true\n" > /etc/cockpit/cockpit.conf
          systemctl daemon-reload
          systemctl restart sshd
          if [[ ! -f ~/.ssh/id_rsa ]]
          then
            ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
          fi

	  rm -rf /dev/shm/yumdb

          if [[ #{num} -eq 1 ]]
          then
            for i in {10..#{node_count+10}}
            do
              echo Setting passwordless ssh to root on 192.168.250.${i}
              sshpass -p foobar ssh-copy-id root@192.168.250.${i} -o StrictHostKeyChecking=no
            done
          fi
        SHELL
        end
      end
    end
