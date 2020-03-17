$script = <<SCRIPT
apt update
apt -y install mongodb wget git
echo "wget"
wget -q https://storage.googleapis.com/golang/getgo/installer_linux
chmod +x installer_linux
./installer_linux
source ~/.bash_profile
rm -f installer_linux
go get -u -v "github.com/gorilla/mux"
go get -u -v "golang.org/x/net/http2"
go get -u -v "golang.org/x/sys/unix"

sh -c "cat << EOF > /etc/systemd/network/99-free5gc.netdev
[NetDev]
Name=uptun
Kind=tun
EOF"

sudo systemctl enable systemd-networkd
sudo systemctl restart systemd-networkd

sysctl -n net.ipv6.conf.uptun.disable_ipv6

sudo sh -c "cat << EOF > /etc/systemd/network/99-free5gc.network
[Match]
Name=uptun
[Network]
Address=45.45.0.1/16
Address=cafe::1/64
EOF"

systemctl enable systemd-networkd
systemctl restart systemd-networkd

ip a flush uptun
systemctl restart networking

apt-get -y install net-tools
ifconfig uptun

apt-get -y install autoconf libtool gcc pkg-config git flex bison libsctp-dev libgnutls28-dev libgcrypt-dev libssl-dev libidn11-dev libmongoc-dev libbson-dev libyaml-dev

git clone https://bitbucket.org/nctu_5g/free5gc-stage-1.git
cd free5gc-stage-1
autoreconf -iv
./configure --prefix=`pwd`/install
make -j `nproc`
cd support/freeDiameter
./make_certs.sh .
cd ../..
make install

ifconfig eth1 192.188.2.2
sh -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -I INPUT -i uptun -j ACCEPT

sudo sh -c "cat << EOF > /etc/init.d/ngc-network-setup
#!/bin/sh
### BEGIN INIT INFO
# Provides:          ngc-network-setup
# Required-Start:    networkd
# Required-Stop:     networkd
# Default-Start:     networkd
# Default-Stop:      networkd
# Short-Description:
# Description:
#
### END INIT INFO

ifconfig eth1 192.188.2.2
sh -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -I INPUT -i uptun -j ACCEPT
EOF"

chmod 755 /etc/init.d/ngc-network-setup
/etc/init.d/ngc-network-setup

ln -s /etc/init.d/ngc-network-setup /etc/rc3.d/S99ngc-network-setup
ln -s /etc/init.d/ngc-network-setup /etc/rc4.d/S99ngc-network-setup
ln -s /etc/init.d/ngc-network-setup /etc/rc5.d/S99ngc-network-setup

apt -y install curl
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
apt -y install nodejs
cd webui
npm install

SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-18.04"
  config.vm.hostname = "free5gc"
  config.vm.network "public_network", bridge: "en0: Wi-Fi (AirPort)", auto_config: false
  config.vm.network :forwarded_port, guest: 3000, host: 3000
  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 4
  end
  config.vm.provision :shell, :inline => $script
end
