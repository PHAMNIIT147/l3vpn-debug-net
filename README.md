setup_development_vm_guide_inphi_copration_internetional_singapore

# :building_construction: Build VM:
```
wget http://releases.ubuntu.com/18.04.3/ubuntu-18.04.3-live-server-amd64.iso
virt-manager
```
# :wrench: Config adminstor when login root
```
sudo psswd root
```

## upgrade kernel for VM:
```
sudo apt-get install --install-recommends linux-generic-hwe-18.04
sudo apt-get update
sudo apt-get dist-upgrade
sudo reboot
```
# :pencil: Enable console through serial (telnet): 

```
sudo sed -i 's/GRUB_CMDLINE_LINUX=.*/GRUB_CMDLINE_LINUX="console=ttyS0,115200 console=tty0"/' /etc/default/grub
sudo update-grub
```

## :pencil2: Note (do the following to interact with keyboard in serial terminal): in machine information, display -> keyboard: en-us

# :globe_with_meridians: FRR source and compilation and installation:

[FRR](http://docs.frrouting.org/projects/dev-guide/en/latest/building-frr-for-ubuntu1804.html)

## :wrench: Install packages to build the FRR. If some more package miss, please google.
```
sudo apt-get install git autoconf automake libtool make gawk libreadline-dev texinfo
sudo apt-get install pkg-config libpam0g-dev libjson-c-dev bison flex python-pytest
sudo apt-get install libc-ares-dev python3-dev libsystemd-dev python-ipaddress
sudo apt-get install libpcre3-dev libpcre3 doxygen graphviz  python3-sphinx install-info
sudo apt-get install build-essential libsystemd-dev libjson-c-dev libcap-dev
sudo apt install snmpd snmp libsnmp-dev
```


## :wrench: Ensure that the libyang build requirements are met before continuing
```
git clone https://github.com/CESNET/libyang.git
cd libyang
mkdir build; cd build
cmake -DENABLE_LYD_PRIV=ON -DCMAKE_INSTALL_PREFIX:PATH=/usr \
      -D CMAKE_BUILD_TYPE:String="Release" ..
make
sudo make install
```

## :alien: Add FRR user and groups
```
sudo groupadd -r -g 92 frr
sudo groupadd -r -g 85 frrvty
sudo adduser --system --ingroup frr --home /var/run/frr/ \
   --gecos "FRR suite" --shell /usr/sbin/nologin frr
sudo usermod -a -G frrvty frr
```

## :package: Source: 
```
git clone https://github.com/frrouting/frr.git frr
git checkout stable/7.3

cd frr
./bootstrap.sh
./configure \
    --enable-exampledir=/usr/share/doc/frr/examples/ \
    --localstatedir=/var/opt/frr \
    --sbindir=/usr/lib/frr \
    --sysconfdir=/etc/frr \
    --enable-multipath=64 \
    --enable-user=frr \
    --enable-group=frr \
    --enable-vty-group=frrvty \
    --enable-configfile-mask=0640 \
    --enable-logfile-mask=0640 \
    --enable-systemd \
    --enable-snmp
```
## not use fpm any more, so not use the following configuration:
```
./configure \
    --enable-exampledir=/usr/share/doc/frr/examples/ \
    --localstatedir=/var/opt/frr \
    --sbindir=/usr/lib/frr \
    --sysconfdir=/etc/frr \
    --enable-multipath=64 \
    --enable-user=frr \
    --enable-group=frr \
    --enable-vty-group=frrvty \
    --enable-configfile-mask=0640 \
    --enable-logfile-mask=0640 \
    --enable-fpm \
    --enable-systemd \
    --enable-snmp
```
```
make
sudo make instal
sudo echo include /usr/local/lib >> /etc/ld.so.conf
sudo ldconfig
```

## Setup modules and frr deamons:
```
cd ~/frr
sudo su (mode root)
sudo install -m 775 -o frr -g frr -d /var/log/frr
sudo install -m 775 -o frr -g frrvty -d /etc/frr
sudo install -m 640 -o frr -g frrvty tools/etc/frr/vtysh.conf /etc/frr/vtysh.conf
sudo install -m 640 -o frr -g frr tools/etc/frr/frr.conf /etc/frr/frr.conf
sudo install -m 640 -o frr -g frr tools/etc/frr/daemons.conf /etc/frr/daemons.conf
sudo install -m 640 -o frr -g frr tools/etc/frr/daemons /etc/frr/daemons
```
## Start frr
```
sudo install -m 644 tools/frr.service /etc/systemd/system/frr.service
sudo systemctl enable frr
```

## enable more protocol in frr
```
sed -i "s/=no/=yes/g" /etc/frr/daemons
vi (or nano) /etc/frr/daemons
edit sharpd=no
service frr stop
service frr start
ps -ef | grep frr
```

## load mpls modules
```
modprobe mpls_router
modprobe mpls_iptunnel
modprobe mpls_gso
modprobe dummy numdummies=1
cat >/etc/modules-load.d/mpls.conf <<EOF
mpls_router
mpls_iptunnel
mpls_gso
dummy
EOF
```

# Enable kernel routing and enable mpls for interfaces ens8,9,10,11

Running on GNS3

```
cat >/etc/sysctl.d/90-mpls-router.conf <<EOF
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
net.ipv4.conf.all.rp_filter=0
net.mpls.platform_labels=1048575
net.ipv4.tcp_l3mdev_accept=1
net.ipv4.udp_l3mdev_accept=1
net.mpls.conf.lo.input=1
net.mpls.conf.ens4.input=1
net.mpls.conf.ens5.input=1
net.mpls.conf.ens6.input=1
net.mpls.conf.ens7.input=1
net.mpls.conf.ens8.input=1
net.mpls.conf.ens9.input=1
net.mpls.conf.ens10.input=1
EOF
sysctl -p /etc/sysctl.d/90-mpls-router.conf
```

# prevent iser took 2 minute waiting during boot:
```
nano /etc/network/interface
​edit: auto  ens3 => allow-hotplug ens3
```

#prevent problem link down or link could not connecto to internet of Qemu image in GNS:
Edit your qemu router template default like ens{0}: cause the problem. It must be ens{port3}. The ens3 is the default port that connect to internet during your build qemu image.

# change Qemu image in GNS3:

[QEMU](https://www.bernhard-ehlers.de/blog/posts/2017-05-20-gns3-modify-qemu-base-image/)

# make fpm-stub: https://github.com/opensourcerouting/fpm-stub
```
cd fpm-stub
make QUAGGA_DIR=<location-of-quagga-code>
```

# install desktop: not use
[DESKTOP](https://itstillworks.com/install-desktop-ubuntu-server-6780086.html)

# install eclipse:
` apt install -y eclipse-cdt-*`
```
URL=https://www.eclipse.org/downloads/download.php
ECLIPSE=/oomph/epp/oxygen/R/eclipse-inst-linux64.tar.gz
MIRROR=1
wget -q -O eclipse-inst-linux64.tar.gz  "${URL}?file=${ECLIPSE}&mirror_id=${MIRROR}"
tar zxf eclipse-inst-linux64.tar.gz
./eclipse-installer/eclipse-inst
```

# install judy:
`sudo apt-get install -y libjudy-dev`
