389 Directory
All servers should connected to a valid NTP server

**Install Packages**

 yum -y install epel-release
 
 yum -y install ntpdate screen traceroute tcpdump atop wget dstat vim  unzip telnet

**Prepare OS For installation**

Edit hosts file and add hostname with corresponding IP addresss

vi /etc/hosts
x.x.x.x  mgmt.domainname.com mgmt

useradd -r -s /sbin/nologin admin

Edit file "/etc/sysctl.conf" and add the following lines at the bottom:

net.ipv4.tcp_keepalive_time = 300
net.ipv4.ip_local_port_range = 1024 65000
fs.file-max = 64000

vi /etc/security/limits.conf

admin               soft     nofile          8192   
admin               hard     nofile          8192

vi /etc/profile

ulimit -n 8192

vi /etc/pam.d/login

session    required     /lib/security/pam_limits.so

systemctl reboot

**Service Installtion**

Download https://releases.pagure.org/389-ds-base/389-ds-base-1.4.3.3.tar.bz2

Extract  into /opt/

yum install 389-ds openldap-clients

**Service Configuration**
 
setup-ds-admin.pl

- Choose a setup type: 2 Typical
- Computer name [core01.yourdomain.com]: [enter]
- System User [nobody]: admin
- System Group [nobody]: admin
- Do you want to register this software with an existing
configuration directory server? [no]: [enter]
- administrator ID [admin]: [enter]
- password: somethingsafe
- Administration Domain [core01.domain.com]: [enter]
- Directory server network port [389]: [enter]
- Directory server identifier [core01]: [enter]
- Suffix [dc=domain, dc=com]: [enter] (Check your suffix if you already have)
- Directory Manager DN [cn=Directory Manager]: [enter]
- Password: somthingsafe
- Administration port [9830]: [enter]
- Are you ready to set up your servers? [yes]:[enter]

Start and enable service in startup
systemctl daemon-reload
systemctl enable --now dirsrv@instance.service (instance name is the host name which is in our document is core01)
systemctl enable --now dirsrv-admin.service

Check service status
 systemctl status 
 systemctl status dirsrv-admin.service

Verify installation
 ldapsearch -x -b "dc=yourdomain,dc=com"

**389 Directory Clustering**

In this step we're going to configure Directory Multi-Master an ReadOnly Replication

For Each Master (Core) Servers :

Log into 389-console, double click on directory server , in configuration tab click on replication

changelog --> enable

changelog database directory --> use default

max changelog records --> unlimited

max changelog age --> 8 days

Now a users must be created to perform replication : uid=Replication Manager,cn=config

Log into 389-console, double click on directory server, in configuration tab under Replication click on userRoot:

Enable Replica --> enable

Replica Role --> Multiple Master

Common Settings --> Replica ID --> Assign a Unique Number

Common Settings --> Purge Delay --> 1 Days

Update Settings --> Current Supplier DNs --> uid=Replication Manager,cn=config

Now Right Click on userRoot and Select New Replication Agreement :

Enter Name and Description and Click Next

on Source and Destination Page Click on Other and enter hostname and port (default is 389).

for authentication mechanism select simple bind and the following information :

Bind as: uid=Replication Manager,cn=config

Password: <Password>

on Replicated Attributes Page Enable Fractional Replication and add memberOf to Excluded attributes.

on Replication Schedule Page Select Always keep directories in sync.

on Initialize Consumer page Select Initialize consumer now

to verify applied settings select newly created agreement name under userRoot and check the Status section.

**Setup Consumer Replicas :**

For Each Replica Server :

First a users must be created to perform replication : uid=Replication Manager,cn=config

Log into 389-console, double click on directory server , in configuration tab click on userRoot under Replication :

Enable Replica --> Enabled

Replica Role --> Dedicated Consumer

Purge delay --> 1 Days

Update Settings --> Current Supplier DNs --> uid=Replication Manager,cn=config

Now on All Master Servers a Replication Agreement Must be Created :

on master servers Right Click on userRoot and Select New Replication Agreement :

Enter Name and Description and Click Next

on Source and Destination Page Click on Other and enter hostname and port (default is 389).

for authentication mechanism select simple bind and the following information :

Bind as: uid=Replication Manager,cn=config

Password: <Password>

