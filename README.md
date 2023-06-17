

sudo apt update
sudo apt install qemu-user-static binfmt-support debootstrap


sudo rm -rf debian-zybo
sudo mkdir debian-zybo
sudo debootstrap --arch=armhf --foreign bookworm debian-zybo
sudo cp /usr/bin/qemu-arm-static debian-zybo/usr/bin/


sudo chroot debian-zybo


##########以下命令都是在chroot中执行###############

export LANG=C
/debootstrap/debootstrap --second-stage

cat > /etc/fstab << EOF
UUID=C579-5B55  /boot  vfat    defaults    0   2
#UUID=ddb12389-4bf7-4dda-8fc0-660de97d4da7 /rootfs ext4  defaults  0 0
EOF
cat /etc/fstab 

cat  > /etc/apt/sources.list <<EOF
deb http://deb.debian.org/debian bookworm main contrib non-free
deb-src http://deb.debian.org/debian bookworm main contrib non-free
deb http://security.debian.org/ stable-security/updates main contrib non-free
deb-src http://security.debian.org/ stable-security/updates main contrib non-free
deb http://deb.debian.org/debian bookworm-updates main contrib non-free
deb-src http://deb.debian.org/debian bookworm-updates main contrib non-free
EOF
cat  /etc/apt/sources.list


apt update
apt install locales dialog
dpkg-reconfigure locales

apt install lsb-release gcc vim openssh-server ntp ntpdate sudo ifupdown resolvconf net-tools udev iputils-ping wget dosfstools unzip binutils libatomic1 v4l-utils yavta curl snapd  lm-sensors software-properties-common
curl -s https://packagecloud.io/install/repositories/ookla/speedtest-cli/script.deb.sh |  bash
apt-get install speedtest

passwd

cat > /etc/network/interfaces << EOF
source /etc/network/interfaces.d/*
auto end0
allow-hotplug end0
iface end0 inet static
  address 192.168.0.33
  netmask 255.255.255.0
  gateway 192.168.0.1
iface end0 inet6 dhcp
EOF
cat  /etc/network/interfaces 


echo Zybo-Z7-20 > /etc/hostname
cat /etc/hostname

cat >  /etc/hosts << EOF
127.0.1.1 Zybo-Z7-20 zybo
EOF
cat   /etc/hosts 


exit

cd ..

########################到此为止#####################

sudo rm debian-zybo/usr/bin/qemu-arm-static

sudo rm -rf /media/yongsh/rootfs/*
sudo cp -r ~/debian-zybo/* /media/yongsh/rootfs/ 
sync



###将SDcard 插到Zybo,以root用户登陆


useradd -m yongsh
usermod --shell /bin/bash yongsh
passwd yongsh


usermod -aG sudo yongsh
chown root:root /usr/bin/sudo
chmod 4755 /usr/bin/sudo

sudo dpkg-reconfigure tzdata
ntpdate  pool.ntp.org

####################################################################


cd /media/yongsh/rootfs
sudo tar cvf debian116back.tar .
sudo mv debian116back.tar ~
cd ~
sync


sudo tar -xvf debian116back.tar -C /media/yongsh/rootfs
sudo tar -xvf debian116back.tar -C ~/debian-zybo
sync 

sudo cp -r ~/debian-zybo/* /media/yongsh/rootfs
sync