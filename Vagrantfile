# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
  :lvm => {
        :box_name => "centos/7",
        :box_version => "1804.02",
        :ip_addr => '192.168.11.101',
    :disks => {
        :sata1 => {
            :dfile => home + '/VirtualBox VMs/sata1.vdi',
            :size => 10240,
            :port => 1
        },
        :sata2 => {
            :dfile => home + '/VirtualBox VMs/sata2.vdi',
            :size => 2048, # Megabytes
            :port => 2
        },
        :sata3 => {
            :dfile => home + '/VirtualBox VMs/sata3.vdi',
            :size => 1024, # Megabytes
            :port => 3
        },
        :sata4 => {
            :dfile => home + '/VirtualBox VMs/sata4.vdi',
            :size => 1024,
            :port => 4
        }
    }
  },
}

Vagrant.configure("2") do |config|

    config.vm.box_version = "1804.02"
    MACHINES.each do |boxname, boxconfig|
  
        config.vm.define boxname do |box|
  
            box.vm.box = boxconfig[:box_name]
            box.vm.host_name = boxname.to_s
  
            #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset
  
            box.vm.network "private_network", ip: boxconfig[:ip_addr]
  
            box.vm.provider :virtualbox do |vb|
                    vb.customize ["modifyvm", :id, "--memory", "256"]
                    needsController = false
            boxconfig[:disks].each do |dname, dconf|
                unless File.exist?(dconf[:dfile])
                  vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                                  needsController =  true
                            end
  
            end
                    if needsController == true
                       vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                       boxconfig[:disks].each do |dname, dconf|
                           vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                       end
                    end
            end
        # hw 3.1
        box.vm.provision "shell", inline: <<-'SHELL'
            mkdir -p ~root/.ssh
            cp ~vagrant/.ssh/auth* ~root/.ssh
            yum install -y mdadm smartmontools hdparm gdisk xfsdump
	    pvcreate /dev/sdb
	    vgcreate vg_tmproot /dev/sdb
	    lvcreate -l +100%FREE -n lv_tmproot /dev/vg_tmproot
	    mkfs.xfs /dev/vg_tmproot/lv_tmproot
	    mkdir /mnt/tmproot
	    mount /dev/vg_tmproot/lv_tmproot /mnt/tmproot
	    xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt/tmproot
	    for f in proc dev sys run boot; do mount --bind /$f /mnt/tmproot/$f ; done
	    chroot /mnt/tmproot grub2-mkconfig -o /boot/grub2/grub.cfg
	    dracut -f /boot/initramfs-$(uname -r).img $(uname -r)
	    sed -i 's/rd.lvm.lv=VolGroup00\/LogVol00/rd.lvm.lv=vg_tmproot\/lv_tmproot/' /boot/grub2/grub.cfg
          SHELL
	
	box.vm.provision "shell", reboot: true
	box.vm.provision "shell", inline: <<-'SHELL'
	    lvremove /dev/VolGroup00/LogVol00 -y
	    lvcreate -L 8G -n LogVol00 /dev/VolGroup00
	    mkfs.xfs /dev/VolGroup00/LogVol00
	    mkdir /mnt/newroot
	    mount /dev/VolGroup00/LogVol00 /mnt/newroot
	    xfsdump -J - /dev/vg_tmproot/lv_tmproot | sudo xfsrestore -J - /mnt/newroot/
	    for i in proc dev sys run boot; do mount --bind /$i /mnt/newroot/$i ;done
	    chroot /mnt/newroot/ grub2-mkconfig -o /boot/grub2/grub.cfg
	    dracut -f /boot/initramfs-$(uname -r).img $(uname -r)
	  SHELL
	
	box.vm.provision "shell", reboot: true
	box.vm.provision "shell", inline: <<-'SHELL'
	    lvremove -y /dev/vg_tmproot/lv_tmproot
	    lvremove -y /dev/vg_tmproot
	  SHELL
	# hw 3.2
	box.vm.provision "shell", inline: <<-'SHELL'
	    lvcreate -L 2G -n LogVol02home /dev/VolGroup00
	    mkfs.xfs /dev/VolGroup00/LogVol02home
	    mkdir /mnt/newhome
	    mount /dev/VolGroup00/LogVol02home /mnt/newhome
	    cp --preserve=context -rfp /home/* /mnt/newhome
	    rm -rf /home/*
	    umount /mnt/newhome
	    mount /dev/VolGroup00/LogVol02home /home/
	    echo 'UUID='`blkid /dev/VolGroup00/LogVol02home -s UUID -o value`'`/home	xfs	defaults	0 0' >> /etc/fstab
	  SHELL
	# hw 3.3
	box.vm.provision "shell", inline: <<-'SHELL'
	    pvcreate /dev/sdd /dev/sde
	    vgcreate VolGroup01 /dev/sdd /dev/sde
	    lvcreate -L 900M -m 1 -n LogVol03var /dev/VolGroup01
	    mkfs.xfs /dev/VolGroup01/LogVol03var
	    mkdir /mnt/newvar
	    mount /dev/VolGroup01/LogVol03var /mnt/newvar
	    cp --preserve=context -rfp /var/* /mnt/newvar/
	    rm -rf /var/* 2>/dev/null
	    umount /mnt/newvar
	    mount /dev/VolGroup01/LogVol03var /var
	    echo 'UUID='`blkid /dev/VolGroup01/LogVol03var -s UUID -o value`'	/var	xfs	defaults	0 0' >> /etc/fstab
	  SHELL
	end
    end
  end
  
