UML@OpenVZ
===========

Execute the User Mode linux of the lastest linux kernel version for TCP congestion control "BBR" .

OpenVZ Server
------

### Install Ubuntu-14.04-x86_64 Trusty:

    apt-get update
    apt-get install build-essential libncurses5-dev libvdeplug-dev libpcap-dev debootstrap -y

### 1.Download the lastest linux kernel for compiling the User Mode Linux(www.kernel.org)

    wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.9.10.tar.xz
    tar xvf linux-4.9.10.tar.xz
    cd linux-4.9.10
    make defconfig ARCH=um

### 2.Edit the .config file and insert the options for the configuration file.

    CONFIG_BINFMT_ELF=y
    CONFIG_HOSTFS=y
    CONFIG_LBD=y
    CONFIG_BLK_DEV=y
    CONFIG_BLK_DEV_LOOP=y
    CONFIG_STDERR_CONSOLE=y
    CONFIG_UNIX98_PTYS=y
    CONFIG_EXT2_FS=y

### 3.user menuconfig select for the TCP congestion control "BBR" and network device.
    
    make menuconfig ARCH=um or make ARCH=um nconfig

    Networking support  --->  Networking options  --->  TCP: advanced congestion control  --->
    
    <*>BBR TCP (NEW)
    <*> Default TCP congestion control (BBR) 
    
    <*> UML Network Devices

### 4.compile the vmlinux file.

    make ARCH=um vmlinux -j4

### 5.start debootstrap for ubuntu14.04 minimal container and install the package.
    
    debootstrap --arch amd64 trusty ./ubuntu1404 http://ftp.ubuntu.com/ubuntu/
    chroot ubuntu1404 /bin/bash
    apt-get update
    apt-get install ethtool traceroute dnsutils vim -y

    vim /etc/network/interfaces    
    auto lo
    iface lo inet loopback
    auto eth0
    iface eth0 inet static
    address 10.0.0.2
    netmask 255.255.255.0
    gateway 10.0.0.1
    
        
