sudo yum install epel-release -y

sudo yum update -y

sudo shutdown -r now

sudo yum install -y java-1.8.0-openjdk

java -version

echo "JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:bin/java::")" | sudo tee -a /etc/profile

source /etc/profile

cd

wget https://archive.apache.org/dist/activemq/5.15.8/apache-activemq-5.15.8-bin.tar.gz

sudo tar -zxvf apache-activemq-5.15.8-bin.tar.gz -C /opt

sudo ln -s /opt/apache-activemq-5.15.8 /opt/activemq

cd /opt/activemq

sudo ./bin/activemq start

vi /usr/lib/systemd/system/activemq.service


[Unit]
Description=activemq message queue
After=network.target
[Service]
PIDFile=/opt/activemq/data/activemq.pid
ExecStart=/opt/activemq/bin/activemq start
ExecStop=/opt/activemq/bin/activemq stop
User=root
Group=root
[Install]
WantedBy=multi-user.target

sudo systemctl enable activemq.service

sudo systemctl start activemq.service

sudo systemctl stop activemq.service
