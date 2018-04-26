# -*- mode: ruby -*-
# vi: set ft=ruby :

$common = <<SCRIPT
#sudo sed -i 's,deb .*archive.ubuntu,deb http://bg.archive.ubuntu,g' /etc/apt/sources.list

# improve somewhat the install process
echo 'APT::Install-Recommends "false";' | sudo tee /etc/apt/apt.conf.d/99no-recommends
echo 'APT::Install-Suggests "false";' | sudo tee -a /etc/apt/apt.conf.d/99no-recommends
echo 'APT::Get::Install-Suggests "false";' | sudo tee -a /etc/apt/apt.conf.d/99no-recommends
echo 'APT::Get::Install-Recommends "false";' | sudo tee -a /etc/apt/apt.conf.d/99no-recommends

# let's get current
sudo apt-get update
sudo DEBIAN_FRONTEND=noninteractive apt-get -y dist-upgrade

# remove some
sudo DEBIAN_FRONTEND=noninteractive apt-get -y autoremove --purge squashfs-tools ubuntu-core-launcher snapd
sudo rm -rf /var/lib/lxd

# make base playground dir
if [ ! -d /data ] ; then
  sudo mkdir /data
fi
SCRIPT

$enc_zfs = <<SCRIPT
# install tools
sudo DEBIAN_FRONTEND=noninteractive apt-get -y install zfsutils-linux

# make encrypted luks
echo cryptoluks | sudo cryptsetup -v --cipher aes-xts-plain64 --hash sha512 --key-file - --use-urandom --iter-time 5000 --batch-mode luksFormat /dev/sdc
echo cryptoluks | sudo cryptsetup -v --key-file - open --type luks /dev/sdc data

# now go for zfs
zpool create -m /data play /dev/mapper/data

# and enable compression
zfs set compression=lz4 play
SCRIPT

$enc_vera = <<SCRIPT
# add repo
if [ ! -r /etc/apt/sources.list.d//unit193-ubuntu-encryption-xenial.list ] ; then
  sudo add-apt-repository ppa:unit193/encryption
  sudo apt-get update
fi

# install
sudo DEBIAN_FRONTEND=noninteractive apt-get -y install veracrypt cryptsetup

# if second run unmount everything
sudo veracrypt --dismount

# wipe disk
sudo dd if=/dev/zero of=/dev/sdc bs=1M count=100

# create normal volume on the whole disk with options
sudo veracrypt --text \
  --create /dev/sdc \
  --encryption aes \
  --filesystem ext4 \
  --hash sha512 \
  --password veracrypt \
  --quick  \
  --keyfiles '' \
  --volume-type normal \
  --random-source /dev/urandom \
  --pim 2000 \
  --non-interactive

# mount
sudo veracrypt --text \
  --mount /dev/sdc /data \
  --password veracrypt \
  --pim 2000 \
  --keyfiles '' \
  --fs-options rw,noatime,nodiratime \
  --non-interactive
SCRIPT

$enc_luks = <<SCRIPT
# setup header
echo cryptoluks | sudo cryptsetup -v --cipher aes-xts-plain64 --hash sha512 --key-file - --use-urandom --iter-time 5000 --batch-mode luksFormat /dev/sdc

# open device and create node
echo cryptoluks | sudo cryptsetup -v --key-file - open --type luks /dev/sdc data

# format with fs and mount
sudo mkfs.ext4 /dev/mapper/data
sudo mount -o rw,noatime,nodiratime /dev/mapper/data /data
SCRIPT

Vagrant.configure("2") do |config|
  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/xenial64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = false

  # define boxes
  # Encrypted zfs + lxd
  config.vm.define "enc-zfs" do |z|
    z.vm.provision "shell", inline: $common
    z.vm.provision "shell", inline: $enc_zfs
    z.vm.hostname = "encryption-zfs-lxd"
    z.vm.provider :virtualbox do |vb|
      vb.linked_clone = true
      vb.memory = "512"
      vb.cpus = "1"
      
      # attach a second disk
      disk = './enc-zfs-disk2.vdi'
      unless File.exist?(disk)
        vb.customize ['createhd', '--filename', disk, '--format', 'VDI', '--size', 4 * 1024]
      end
      vb.customize ['storageattach', :id, '--storagectl', 'SCSI', '--port', 6, '--device', 0, '--type', 'hdd', '--medium', disk]
    end
  end

  # Veracrypt
  config.vm.define "enc-vera" do |v|
    v.vm.provision "shell", inline: $common
    v.vm.provision "shell", inline: $enc_vera
    v.vm.hostname = "encryption-vera"
    v.vm.provider :virtualbox do |vb|
      vb.linked_clone = true
      vb.memory = "512"
      vb.cpus = "1"

      # attach a second disk
      disk = './enc-vera-disk2.vdi'
      unless File.exist?(disk)
        vb.customize ['createhd', '--filename', disk, '--format', 'VDI', '--size', 4 * 1024]
      end
      vb.customize ['storageattach', :id, '--storagectl', 'SCSI', '--port', 6, '--device', 0, '--type', 'hdd', '--medium', disk]
    end
  end
 
  # luks
  config.vm.define "enc-luks" do |l|
    l.vm.provision "shell", inline: $common
    l.vm.provision "shell", inline: $enc_luks
    l.vm.hostname = "encryption-luks"
    l.vm.provider :virtualbox do |vb|
      vb.linked_clone = true
      vb.memory = "512"
      vb.cpus = "1"
      
      # attach a second disk
      disk = './enc-luks-disk2.vdi'
      unless File.exist?(disk)
        vb.customize ['createhd', '--filename', disk, '--format', 'VDI', '--size', 4 * 1024]
      end
      vb.customize ['storageattach', :id, '--storagectl', 'SCSI', '--port', 6, '--device', 0, '--type', 'hdd', '--medium', disk]
    end
  end
end
