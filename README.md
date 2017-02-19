UML@OpenVZ
===========

Execute the User Mode linux of the lastest linux kernel version for TCP congestion control "BBR" .

OpenVZ Server
------

### Install Ubuntu-14.04-x86_64 Trusty:

    apt-get update
    apt-get install tmux build-essential libncurses5-dev libvdeplug-dev libpcap-dev debootstrap -y

### Download the lastest linux kernel for compiling the User Mode Linux(www.kernel.org)

    wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.9.10.tar.xz
    tar xvf linux-4.9.10.tar.xz
    cd linux-4.9.10
    make defconfig ARCH=um

### Edit the .config file and insert the options for the configuration file.

    CONFIG_BINFMT_ELF=y
    CONFIG_HOSTFS=y
    CONFIG_LBD=y
    CONFIG_BLK_DEV=y
    CONFIG_BLK_DEV_LOOP=y
    CONFIG_STDERR_CONSOLE=y
    CONFIG_UNIX98_PTYS=y
    CONFIG_EXT2_FS=y

### Use menuconfig select for the TCP congestion control "BBR" and network device.
    
    make menuconfig ARCH=um or make ARCH=um nconfig

    Networking support  --->  Networking options  --->  TCP: advanced congestion control  --->
    
    <*>BBR TCP (NEW)
    <*> Default TCP congestion control (BBR) 
    
    <*> UML Network Devices

### Compile the vmlinux file with 4 Thread.

    make ARCH=um vmlinux -j4

### Use debootstrap for building the ubuntu14.04 container and install the package.
    
    debootstrap --arch amd64 trusty ./ubuntu1404 http://ftp.ubuntu.com/ubuntu/
    chroot ubuntu1404 /bin/bash
    apt-get update
    apt-get install git -y

### Assign the ip address for the container eth0.

    vi /etc/network/interfaces
    auto eth0
    iface eth0 inet static
    address 10.0.0.2
    netmask 255.255.255.0
    gateway 10.0.0.1

### For login the container to change the tty0 and securetty config. 

    #create the tty0 config and change the config.
    cp /etc/init/tty1.conf /etc/init/tty0.conf
    vi /etc/init/tty0.conf
    exec /sbin/getty -8 38400 tty0

    #insert the tty0 enable root login
    vi /etc/securetty
    tty0

    #change passwd for root
    passwd root

    #if you need the openssh-server, it need to permit root login.
    apt-get install openssh-server
    vi /etc/ssh/sshd_config
    PermitRootLogin yes

### Install the shadowsocksr to the container and add the user.

    cd /usr/local && git clone https://github.com/shadowsocksr/shadowsocksr.git
    cd shadowsocksr && bash initcfg.sh
    vi userapiconfig.py
    API_INTERFACE = 'mudbjson'
    python mujson_mgr.py -a -u ssr443 -p 443 -m aes-128-cfb -k ssr-passwd -O auth_aes128_sha1 -o tls1.2_ticket_auth
    
    vi /etc/rc.local
    /usr/local/shadowsocksr/run.sh
    
    exit

## Set up the Host SSH tcp port 22 for bypass and NAT traffic for container.
    
    iptables -t nat -A PREROUTING -p tcp -m tcp --dport 22 -j RETURN
    iptables -t nat -A PREROUTING -d $(hostname -i)/32 -j DNAT --to-destination 10.0.0.2
    iptables -t nat -A POSTROUTING -d 10.0.0.2/32 -j SNAT --to-source 10.0.0.1
    iptables -t nat -A POSTROUTING -s 10.0.0.0/24 ! -d 10.0.0.0/24 -j MASQUERADE

## Set up the TAP for Host and Run the vmlinux
    ip tuntap add tap0 mode tap
    ip addr add 10.0.0.1/24 dev tap0
    ip link set tap0 up  
    mount -o remount,size=256m /dev/shm
    dd if=/dev/zero of=rootfs.img bs=1MB count=1000
    /root/vmlinux root=/dev/ubda rootfstype=hostfs hostfs=/root/ubuntu1404 ubd0=/root/rootfs.img eth0=tuntap,tap0 mem=256m rw
~
