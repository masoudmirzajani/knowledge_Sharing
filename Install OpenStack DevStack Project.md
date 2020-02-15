Install OpenStack DevStack Project
========================

**1- install ubuntu server  for 4 vm "contoller1, compute1, blockstorage1, objectstorage1"**
**2- install chrony  && add all vm's hostname and address on hosts file on all machine**

apt update && apt install chrony
vim /etc/hosts

**3- on controller vm**  
vim /etc/chrony/chrony.conf 
Allow 192.168.122.0/24 
systemctl restart chrony

**4- on other vm's** 
vim /etc/chrony/chrony.conf 
server "controller address""  iburst
comment pool...
systemctl restart chrony

**5- use below manuall page for all vm's https://docs.openstack.org/devstack/latest/**

$ sudo useradd -s /bin/bash -d /opt/stack -m stack
$ echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
$ sudo su - stack
$ git clone https://opendev.org/openstack/devstack
$ cd devstack

Create a local.conf 
[[local|localrc]]
ADMIN_PASSWORD=stack
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD

./stack.sh



**6- install below on controller**
apt install software-properties-common
add-apt-repository cloud-archive:pike
apt update && apt dist-upgrade
reboot
apt install python-openstackclient

dpkg -l
dpkg -l |grep -i python-nova
dpkg -l |grep -i python-glance
dpkg -l |grep -i python-neutron

**7- install DB on controller***
apt install mariadb-server python-mysqldb
vim /etc/mysql/mariadb.conf.d/99-openstack.cnf

[mysqld]

bind-address = controller address

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
charecter-set-server = utf8

systemctl restart mysql

mysql_secure_installation

**For keystone Project**
mysql -u root -p
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'stack';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'stack';
\q

**For Glance Project**
mysql -u root -p
CREATE DATABASE glance;

**8- install rabbitmq on controller***
apt install rabbitmq-server
rabbitmqctl add_user stack stack
rabbitmqctl set_permissions stack ".*"  ".*"  ".*" 

rabbitmqctl list_users
rabbitmqctl list_user_permissions stack 

**9- install memcached on controller***
apt install memcached python-memcache 
vim /etc/memcached.conf
-l controller1 address
systemctl restart memcached 


**10- Install the etcd on controller**

apt install etcd
vim /etc/default/etcd

ETCD_NAME="controller1"
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER="controller=http://controller_addr:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://controller_addr:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://controller_addr:2379"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://controller_addr:2379"

systemctl enable etcd
systemctl restart etcd


**11- Install keystone Project**

apt install keystone spache2 libapace2-mod-wsgi
ss -ntl
vim /etc/keystone/keystone.conf
[database]
connection = mysql+pymysql://keystone:stack@controller1/keystone
[token]
provider = fernet

su -s /bin/sh -c "keystone-manage db_sync" keystone

keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone

keystone-manage bootstrap --bootstrap-password stack \
  --bootstrap-admin-url http://controller1:5000/v3/ \
  --bootstrap-internal-url http://controller1:5000/v3/ \
  --bootstrap-public-url http://controller1:5000/v3/ \
  --bootstrap-region-id RegionOne

vim /etc/sysconfig/apache2 

APACHE_SERVERNAME="controller1"

vim  /etc/apache2/conf.d/wsgi-keystone.conf

Listen 5000

<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>


chown -R keystone:keystone /etc/keystone
systemctl enable apache2.service
systemctl start apache2.service

export OS_USERNAME=admin
export OS_PASSWORD=stack
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://controller1:5000/v3
export OS_IDENTITY_API_VERSION=3


openstack project create --domain default --description "Service Project" service
openstack project create --domain default --description "Demo Project" demo
openstack user create --domain default --password-prompt demo
pass: stack
openstack role create user
openstack role add --project demo  --user demo user

unset OS_AUTH_URL OS_PASSWORD

openstack --os-auth-url http://controller1:5000/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name admin --os-username admin token issue


openstack --os-auth-url http://controller1:5000/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name demo --os-username demo token issue

vim admin-openrc

export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=stack
export OS_AUTH_URL=http://controller1:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2

vim demo-openrc

export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OSPASSWORD=stack
export OS_AUTH_URL=http://controller1:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2


. admin-openrc
openstack token issue

openstack service list

