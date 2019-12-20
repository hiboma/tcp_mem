# frozen_string_literal: true
# -*- mode: ruby -*-

$proxy_script = <<~SCRIPT

set -x

sudo sed -i -e "s#/archive.ubuntu.com#/jp.archive.ubuntu.com#g" /etc/apt/sources.list
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install -y --install-recommends linux-image-generic linux-headers-generic
sudo apt install -y haproxy gawk wget git diffstat unzip texinfo gcc-multilib \
build-essential chrpath socat libsdl1.2-dev xterm libncurses5-dev \
             lzop flex libelf-dev kmod
sudo snap install --devmode bpftrace
sudo snap connect bpftrace:system-trace

grep -q 192.168.100.200 /etc/haproxy/haproxy.cfg
if [ $? == 1 ]; then
  cat <<EOS >>/etc/haproxy/haproxy.cfg
listen http-in
    bind *:8888
    server server000 192.168.100.200:80 maxconn 32
EOS
fi
SCRIPT

$server_script = <<~SCRIPT
set -x
sudo sed -i -e "s#/archive.ubuntu.com#/jp.archive.ubuntu.com#g" /etc/apt/sources.list
sudo apt-get update
sudo apt-get install -y nginx
sudo dd if=/dev/urandom of=/var/www/html/1byte.txt  bs=1    count=1
sudo dd if=/dev/urandom of=/var/www/html/1mb.txt    bs=1024 count=1024
sudo dd if=/dev/urandom of=/var/www/html/10mb.txt   bs=1024 count=1024x10
sudo dd if=/dev/urandom of=/var/www/html/1000mb.txt bs=1024 count=1024x1024
SCRIPT

Vagrant.configure('2') do |config|

  # curl 入っていれば ok
  config.vm.define 'client' do |c|
    c.vm.box      = 'bento/ubuntu-18.04'
    c.vm.hostname = 'client000'
    c.vm.network :private_network, ip:"192.168.100.2"
  end

  # haproxy
  # - sysctl net.ipv4.tcp_mem='1 2 3'
  config.vm.define 'proxy' do |c|
    c.vm.box      = 'bento/ubuntu-18.04'
    c.vm.hostname = 'proxy000'
    c.vm.provision 'shell', inline: $proxy_script
    c.vm.network :private_network, ip:"192.168.100.100"
  end

  # nginx
  config.vm.define 'server' do |c|
    c.vm.box      = 'bento/ubuntu-18.04'
    c.vm.hostname = 'server000'
    c.vm.provision 'shell', inline: $server_script
    c.vm.network :private_network, ip:"192.168.100.200"
  end

  config.ssh.insert_key = false
  config.vm.provider 'virtualbox' do |vb|
    vb.gui = true
    vb.customize ['modifyvm', :id, '--memory', '512', '--cpus', '2', '--ioapic', 'on']
  end
end
