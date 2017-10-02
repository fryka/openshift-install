# openshift-install

https://github.com/jorgemoralespou/s2i-java

oc create -f https://raw.githubusercontent.com/jorgemoralespou/s2i-java/master/ose3/s2i-java-imagestream.json



https://www.youtube.com/watch?v=-OOnGK-XeVY

 

Installation of OS and packages takes roughly 16GB and you will need 40GB free for OpenShift not to complain 
 

 

We want the VM to be reachable on our local network and not be on a separate subnet so select bridged adapter
 

Download start-up image from here http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1708.iso
 

Run the installation and remember to enable the network (by default it is off)

After first boot stop the VM and add an additional disk which will be used for the docker repository. Docker production systems do not like to use a loopback devicemapper (run “docker info”, after it’s installed, and see for yourself)

 

 

yum update -y && yum install -y epel-release yum-utils device-mapper-persistent-data lvm2 git wget ansible python-cryptography pyOpenSSL.x86_64 java
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker
service docker stop

vi /etc/lvm/profile/docker-thinpool.profile

activation {
  thin_pool_autoextend_threshold=80
  thin_pool_autoextend_percent=20
}

pvcreate /dev/sdb
<br/>
vgcreate docker /dev/sdb
<br/>
lvcreate --wipesignatures y -n thinpool docker -l 95%VG
<br/>
lvcreate --wipesignatures y -n thinpoolmeta docker -l 1%VG
<br/>
lvconvert -y --zero n -c 512K --thinpool docker/thinpool --poolmetadata docker/thinpoolmeta
<br/>
lvchange --metadataprofile docker-thinpool docker/thinpool
<br/>
lvs -o+seg_monitor
<br/>
rm -rf /var/lib/docker

vi /etc/docker/daemon.json

{
    "storage-driver": "devicemapper",
    "storage-opts": [
    "dm.thinpooldev=/dev/mapper/docker-thinpool",
    "dm.use_deferred_removal=true",
    "dm.use_deferred_deletion=true"
    ]
}

<br/>
service docker start
<br/>
git clone http://github.com/openshift/openshift-ansible
<br/>
git clone http://github.com/gshipley/installcentos
<br/>

vi installcentos/inventory.erb
Change version to 3.6.0
Remove cert entries (2rows) and update host names
Add an [etcd] section with one entry for the consol.[hostname}

vi /etc/hosts
Add entry for 127.0.0.1 and the console.[hostname]

ssh-keygen -t rsa
ssh-copy-id root@console.bfcglobal.com
ansible-playbook -i ./installcentos/inventory.erb openshift-ansible/playbooks/byo/config.yml


htpasswd -b /etc/origin/master/htpasswd mathiasf password

update the windows hosts file in C:\Windows\System32\drivers\etc

(https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/#configure-direct-lvm-mode-for-production)
