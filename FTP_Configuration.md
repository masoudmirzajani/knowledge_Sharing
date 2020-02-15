Create FTP Acount Steps
1. Install vsftpd and any required packages
yum install vsftp* -y
2. Edit the configuration file for vsftpd
 vi /etc/vsftp/vsftp.conf
anonymous_enable=NO

local_enable=YES

write_enable=YES

local_umask=022

dirmessage_enable=YES

xferlog_enable=YES

connect_from_port_20=YES

xferlog_std_format=YES

chroot_local_user=YES

local_root=/var/www/GandouAppList/public

chmod_enable=NO

listen=YES

pam_service_name=vsftpd

userlist_enable=YES

tcp_wrappers=YES

3. Create ftp user and ftp group
groupadd ftpgroup

useradd -g ftpgroup -s /usr/sbin/nologin -m -d /var/www/GandouAppList/public ftpuser

passwd ftpuser
4. Set permission, group and owner of files and directories
cd /var/www/GandouAppList/public

chown –R ftpuser .

chgrp -R ftpgroup .

find . -type f -exec chmod 0444 {} \;

find . -type d -exec chmod 0777{} \;
5. Start service
systemctl start vsftpd

systemctl enable vsftpd
5. Check firewall (firewalld or iptables)
firewall-cmd --permanent --add-port=21/tcp
firewall-cmd --permanent --add-service=ft
