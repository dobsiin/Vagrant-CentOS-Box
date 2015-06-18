#####################
# Vagrant-CentOS-Box
based on config.vm.box = "hp-ess/docker" 
create folder vagrant with cntlm.rmp in it or use default shared folder ./vagrant.
Box with cntlm, docker, docker-compose and vboxguestadditions
#####################

#####################
# install cntlm
#####################
cd /opt/cntlm
sudo rpm -i cntlm.rpm
sudo rm /etc/cntlm.conf
sudo vi /etc/cntlm.conf
sudo cp -uv cntlm.conf /etc
sudo cntlm
#####################
# make cntlmd start on startup
#####################
sudo /sbin/chkconfig --add cntlmd
sudo /sbin/chkconfig --list cntlmd
sudo /sbin/chkconfig cntlmd on
#####################
# yum proxy 
#####################
nothing to do here, because Vagrantfile took care of it
#####################
# vbox guestadd install
#####################
# dependency for vbox guestadd
sudo yum -y update
sudo yum-y install kernel-devel gcc
# install vbox guestadd
cd /opt
sudo wget -c http://download.virtualbox.org/virtualbox/4.3.26/VBoxGuestAdditions_4.3.26.iso -O VBoxGuestAdditions_4.3.26.iso
sudo mount VBoxGuestAdditions_4.3.26.iso -o loop /mnt
cd /mnt
sudo sh VBoxLinuxAdditions.run --nox11
cd /opt
sudo rm *.iso
#####################
# reinstall docker, because it is broken by yum update
#####################
sudo yum -y remove docker
sudo yum -y install docker
sudo service docker start
sudo chkconfig docker on
# enable proxy for docker daemon
sudo mkdir /etc/systemd/system/docker.service.d
sudo vi /etc/systemd/system/docker.service.d/http-proxy.conf
# instert this into http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://172.17.42.1:3128"
# reload and restart
sudo systemctl daemon-reload
sudo systemctl restart docker
# pull first image to test
sudo docker pull centos
#####################
# create Base Image 158.226.125.18:5000:centos:v1
#####################
sh /root/Container/centos/createBaseImage.sh
#####################
# install docker-compose
#####################
sudo -i
curl -L https://github.com/docker/compose/releases/download/1.2.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
exit
####################
# enable private registry access
####################
sudo vi /etc/sysconfig/docker
# instert this to docker
ADD_REGISTRY='--add-registry 158.226.125.18:5000'
INSECURE_REGISTRY='--insecure-registry 158.226.125.18:5000'
# restart docker
sudo systemctl restart docker.service
###################
# create .box
###################
vagrant package --output centox.box
# add the box to vagrant
vagrant box add centosbox centos.box
# create a new Vagrantfile inside a dir
vagrant init centosbox
###################
# putty ssh
###################
user: vagrant
pass: vagrant
