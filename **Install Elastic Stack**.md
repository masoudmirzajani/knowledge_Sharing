**Install Elastic Stack**

Prerequisites
oracle JAVA is recommended
enable EPEL

Install and configure Elastic search
Main Installation Documentation

Method 1: Online
Download and install the public signing key:

rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
Add the following in your /etc/yum.repos.d/ directory in a file with a .repo suffix, for example elasticsearch.repo

[elasticsearch-1.5]
name=Elasticsearch repository for 1.5.x packages
baseurl=http://packages.elastic.co/elasticsearch/1.5/centos
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
And your repository is ready for use. You can install it with:

yum install elasticsearch
EL6 :

sudo service elasticsearch restart
sudo /sbin/chkconfig --add elasticsearch
EL7:

systemctl daemon-reload
systemctl enable elasticsearch.service
systemctl start elasticsearch.service
Method 2: Offline; Preferred
Copy Elastic Search rpm and checksum to server:
elasticsearch-6.2.1.rpm
elasticsearch-6.2.1.rpm.sha512
Verify and Install rpm
shasum -a 512 -c elasticsearch-6.2.1.rpm.sha512 
check if OK
rpm -Uvh elasticsearch-6.2.1.rpm
Verify Installation
Wait, at least a minute to let the Elastic Search get fully restarted, otherwise testing will fail.

curl -X GET http://localhost:9200
Install and Configure Logstash
Main Installation Documentation

Main Running Documentation

Install Logstash RPM (repo is also available), from Aris Gandou Repo

Method 1: Offline; Prefered
Copy logstash rpm and checksum file to server:
logstash-6.2.1.rpm
logstash-6.2.1.rpm.sha512
Verify and Install rpm
shasum -512 -c logstash-6.2.1.rpm.sha512
check if OK
rpm -Uvh logstash-6.2.1.rpm
Start Logstash
sudo systemctl start logstash.service
Verify Logstash
TODO
Install and Configure Kibana
Kidbana provides visualization of logs, download it from official website. Use the following command to download it in terminal.

Main Installation Documentation

Install Kibana RPM, from Aris Gandou Repository

Method 1: Offline; Prefered
Copy kibana rpm and checksum file to server:
kibana-6.2.1-x86_64.rpm
kibana-6.2.1-x86_64.rpm.sha512
Verify and Install rpm
sha512sum -c kibana-6.2.1-x86_64.rpm.sha512
check if OK
rpm -Uvh kibana-6.2.1-x86_64.rpm
Configure Network
by default kibana listens to localhost, uncomment the following line and update it to appropriate ip address
server.host: localhost
Enable and Start Kibana
sudo systemctl start kibana.service
sudo systemctl enable kibana.service
Verify Kibana
at the end kibana should be accessible using http://server_ip_address:5601
Configure Firewall
TODO
OLD DOCS
Configure SSL
4- Create SSL certificate:

Option 1: (Hostname FQDN)

Before creating a certificate, make sure you have A record for logstash server; ensure that client servers are able to resolve the hostname of the logstash server. If you do not have DNS, kindly add the host entry for logstash server; where 10.0.0.26 is the ip address of logstash server and itzgeek is the hostname of your logstash server.

vi /etc/hosts

10.0.0.26 itzgeek Lets create a SSl certificate.

Goto OpenSSL directory.

cd /etc/pki/tls Execute the following command to create a SSL certificate, replace “red” one in with your real logstash server.

openssl req -x509 -nodes -newkey rsa:2048 -days 365 -keyout private/logstash-forwarder.key -out certs/logstash-forwarder.crt -subj /CN=itzgeek

Option 2: (IP Address)

Before creating a SSL certificate, we would require to an add ip address of logstash server to SubjectAltName in the OpenSSL config file.

vi /etc/pki/tls/openssl.cnf
Goto “[ v3_ca ]” section and replace “yellow” one with your logstash server ip.

subjectAltName = IP:10.0.0.26 Goto OpenSSL directory.

cd /etc/pki/tls
Execute the following command to create a SSL certificate.

openssl req -x509 -days 365 -batch -nodes -newkey rsa:2048 -keyout private/logstash-forwarder.key -out certs/logstash-forwarder.crt
This logstash-forwarder.crt should be copied to all client servers those who send logs to logstash server.



Configure Logstash
5- Configure Logstash:

Logstash configuration files can be found in /etc/logstash/conf.d/, just an empty folder. We would need to create file, logstash configuration files consist of three section input, filter and output; all three section can be found either in single file or each section will have separate files ends with .conf.

Here we will use a single file to place an input, filter and output sections.

	# vi /etc/logstash/conf.d/logstash_syslogs.conf
	
In the first section, we will put an entry for input configuration. The following configuration sets lumberjack to listen on port 5050 for incoming logs from the logstash-forwarder that sits in client servers, also it will use the SSL certificate that we created earlier.

		input {
		lumberjack {
		port => 5050
		type => "logs"
		ssl_certificate => "/etc/pki/tls/certs/logstash-forwarder.crt"
		ssl_key => "/etc/pki/tls/private/logstash-forwarder.key"
		}
		}
In the second section, we will put an entry for filter configuration. Grok is a filter in logstash, which does parsing of logs before sending it to Elasticsearch for storing. The following grok filter will look for the logs that are labeled as ‘syslog” and tries to parse them to make a structured index.

		filter {
		if [type] == "syslog" {
			grok {
			  match => { "message" => "%{SYSLOGLINE}" }
			}
		
			date {
		match => [ "timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
		}
		  }
		
		}

Consider visiting grokdebugger for filter patterns.

In the third section, we will put an entry of output configuration. This section defines location where the logs get stored; obviously it should be Elasticsearch.

		output {
		elasticsearch { host => localhost index => "logstash-%{+YYYY.MM.dd}" }
		stdout { codec => rubydebug }
		}

Now start the logstash service.

	# service logstash start
Logstash server logs are stored in the following file, will help us to troubleshoot the issues.

	# cat /var/log/logstash/logstash.log
Next we will configure a logstash-forwarder to ship logs to logstash server.


6- Configure Logstash-forwarder

Logstash-forwarder is a client software which ship logs to a logstash server, it should be installed on all client servers. Logstash-forwarder can be downloaded from official website or you can use the following command to download it in terminal and install it.

	# wget https://download.elasticsearch.org/logstash-forwarder/binaries/logstash-forwarder-0.4.0-1.x86_64.rpm

	# rpm -Uvh logstash-forwarder-0.4.0-1.x86_64.rpm
	
Logstash-forwader uses SSL certificate for validating logstash server identity, so copy the logstash-forwarder.crt that we created earlier from the logstash server to the client. Open up the configuration file.

	# vi /etc/logstash-forwarder.conf
	
In the “network” section, mention the logstash server with port number and path to the logstash-forwarder certificate that you copied from logstash server. This section defines the logstash-forwarder to send a logs to logstash server “itzgeek” on port 5050 and client validates the server identity with the help of SSL certificate.

Note: Replace “itzgeek” with ip address incase if you are using IP SAN.

	"servers": [ "itzgeek:5050" ],

	"ssl ca": "/etc/pki/tls/certs/logstash-forwarder.crt",

	"timeout": 15
	
In the “files” section, configures what all are files to be shipped. In this article we will configure a logstash-forwarder to send a logs (/var/log/messages) to logstash server with “syslog” as type.

	{
	"paths": [
	"/var/log/messages"
	],
	
	"fields": { "type": "syslog" }
	}
	
Restart the service.

	# systemctl start logstash-forwarder.service

You can look at a log file in case of any issue.

	# cat /var/log/logstash-forwarder/logstash-forwarder.err
