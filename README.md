OCS Installation
Install this on Centos 7 Minimal installation

Install wget

yum install wget -y
Download and run ocssetup.sh on the server

wget -O - https://raw.githubusercontent.com/nageshwaran08/OCS-and-GLPI-Installation/master/ocssetup.sh | bash
Launch MySQL Secure Installation script

/usr/bin/mysql_secure_installation
Install additional package

cpan XML::Entities
cpan Apache::DBI
cpan ModPerl::MM
cpan Apache2::SOAP
Exclude ports 80 for Apache, 3306 for MySQL and 25 for SMTP from iptables

firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --zone=public --add-port=25/tcp --permanent
firewall-cmd --reload
Install and activate the REMI and EPEL RPM Repositories

wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm && wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm && rpm -Uvh *.rpm
Update PHP

yum update php* --enablerepo=remi -y
Verify the new PHP version

php -v

PHP 5.4.45 (cli) (built: Mar  1 2018 09:57:11) 
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.4.0, Copyright (c) 1998-2014 Zend Technologies
Download OCS Inventory and run setup

OCS_VER=2.4.1 &&
cd /var/www/html &&
wget https://github.com/OCSInventory-NG/OCSInventory-ocsreports/releases/download/2.4.1/OCSNG_UNIX_SERVER_$OCS_VER.tar.gz &&
tar -xzvf OCSNG_UNIX_SERVER_$OCS_VER.tar.gz &&
cd OCSNG_UNIX_SERVER_$OCS_VER &&
./setup.sh
Increase post_max_size and upload_max_filesize in /etc/php.ini

post_max_size = 200M
upload_max_filesize = 200M
Restart Apache

service httpd restart
Add write permission to the directory

chmod +w /var/lib/ocsinventory-reports
Run mysql_upgrade

mysql_upgrade -uroot -p[password]
Create OCS user in MySQL and assign privileges for OCSWEB database

mysql -uroot -p[password]
GRANT ALL PRIVILEGES ON `ocsweb` .* TO 'ocs'@'localhost' IDENTIFIED BY 'ocs' WITH GRANT OPTION;
Perform initial OCS config then login to OCS

URL: [IP Address]/ocsreports
Login: admin
Password: admin
MySQL login: ocs
MySQL password: [your password when setting up MySQL]
Name of Database: ocsweb
MySQL Hostname: localhost
Change 'Trace Deleted' config to 'ON' in OCS. Config > Config > Server

img

img

Remove install script

rm -f  /usr/share/ocsinventory-reports/ocsreports/install.php
GLPI Installation
Download GLPI

cd /var/www/html
wget https://github.com/glpi-project/glpi/releases/download/9.1.7.1/glpi-9.1.7.1.tgz
tar -xzvf glpi-9.1.7.1.tgz
Update permissions

chown apache:apache -R glpi/
chmod 777 glpi/files/ glpi/config/
Edit /etc/httpd/conf/httpd.conf file. Change all occurrences of AllowOverride None to AllowOverride All

...
<Directory />
    Options FollowSymLinks
    AllowOverride All
</Directory>
...
AllowOverride All
...
Restart Apache

service httpd restart
Setting up database for GLPI use

mysql -u root -p [rootsecret]
CREATE USER 'glpi'@'%' IDENTIFIED BY 'glpisecret';
GRANT USAGE ON *.* TO 'glpi'@'%' IDENTIFIED BY 'glpisecret';
CREATE DATABASE IF NOT EXISTS `glpi`;
GRANT ALL PRIVILEGES ON `glpi`.* TO `glpi`@'%';
CREATE USER 'sync'@'%' IDENTIFIED BY 'syncsecret';
GRANT USAGE ON *.* TO 'sync'@'%' IDENTIFIED BY 'syncsecret';
GRANT SELECT ON `ocsweb`.* TO `sync`@'%';
GRANT DELETE ON `ocsweb`.`deleted_equiv` TO `sync`@`%`;
GRANT UPDATE (`CHECKSUM`) ON `ocsweb`.`hardware` TO `sync`@`%`;
FLUSH PRIVILEGES;
exit
Install GLPI cronjob

crontab -u apache -e
* * * * * /usr/bin/php /var/www/html/glpi/front/cron.php &>/dev/null
Login to GLPI

URL: [IP Address]/glpi
Login: glpi
Password: glpi
MySQL Server: 127.0.0.1
User: glpi
Password: glpisecret
GLPI and OCS Integration
Download OCS plugin, extract and change it's ownership.

cd /var/www/html/glpi/plugins
wget https://github.com/pluginsGLPI/ocsinventoryng/releases/download/1.4.3/glpi-ocsinventoryng-1.4.3.tar.gz
tar -xzvf glpi-ocsinventoryng-1.4.3.tar.gz
chown -R apache:apache ocsinventoryng/
Go to GLPI > Setup > Plugin page. Click Install, then Enable.

Connect GLPI to OCS DB using 'sync' account.

Add OCS Server details

img

Go to Datas to import, from dropdown select YES.

img

OCS Agent Installation
http://wiki.ocsinventory-ng.org/index.php?title=Documentation:UnixAgent

Force Election of Host to do IP Discovery
Select host in OCS.
Go to CONFIGURATION > EDIT > NETWORK SCANS > Select network in IP Discover dropdown
