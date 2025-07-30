# LibreNMS Bandwidth Monitoring
Installation and Configuration of LibreNMS for network bandwidth monitoring

UBUNTU 22.04 : https://bit.ly/ubuntu22045

# System Preparations
Need root access to install it properly on Ubuntu to do this follow the steps below. <br />
Press <code>CTRL + ALT + T</code><br />
then type in
```
sudo -i
```
Then you need to enter your Ubuntu Password<br/>
You will Notice the changes of symbol from "$" to "#"<br/>
means that you gain the root access.<br/>
Next is to update and upgrade
```
sudo apt update && apt upgrade -y
```
# Change the timezone
```
timedatectl set-timezone Asia/Manila
```
# Install Required Packages
```
apt install -y software-properties-common
add-apt-repository universe

apt install -y \
  nginx \
  php-cli php-fpm php-mysql php-snmp php-curl php-gd php-mbstring php-xml php-zip php-bcmath php-tokenizer php-ldap php-common \
  mariadb-server \
  composer \
  fping \
  git \
  graphviz \
  imagemagick \
  mtr-tiny \
  nmap \
  rrdtool \
  snmp \
  snmpd \
  whois \
  unzip \
  curl \
  acl \
  python3-dotenv \
  python3-pymysql \
  python3-redis \
  python3-setuptools \
  python3-systemd \
  python3-pip
```
# Creation of LibreNMS Initial User
```
useradd -r -M -d /opt/librenms librenms
usermod -a -G librenms www-data
```
# Clone LibreNMS
```
cd /opt
git clone https://github.com/librenms/librenms.git
chown -R librenms:librenms /opt/librenms
chmod 771 /opt/librenms
```
# Update Php Version from 8.1 to 8.2
Repository<br />
```
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
```
Installing Php 8.2 and extensions
```bash
sudo apt install -y php8.2 php8.2-cli php8.2-fpm php8.2-common php8.2-curl php8.2-gd php8.2-mysql php8.2-mbstring php8.2-snmp php8.2-xml php8.2-zip php8.2-bcmath php8.2-tokenizer php8.2-ldap
```
# Configuration of MySQL
```bash
mysql -u root -p
```
Note : You may input the password of your Ubuntu again<br/>
Then inside the MySQL Prompt<br/>
```sql
CREATE DATABASE librenms CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'librenms'@'localhost' IDENTIFIED BY 'YourStrongPassword';
GRANT ALL PRIVILEGES ON librenms.* TO 'librenms'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
Note : Replace the <code>YourStrongPasswordwith</code> your preferred Secured Password this will be used upon setting up the webui of LibreNMS<br/>
<br/>
Edit Server config
```
nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
Inside of this file add this at the end of the content
```ini
innodb_file_per_table=1
lower_case_table_names=0
```
Save the config file <code> CTRL + X then y then press Enter </code> to save the edited config file.
# NGINX Configuration
Create the NGINX config file
```
nano /etc/nginx/sites-available/librenms
```
Paste this config.
```
server {
    listen 80;
    server_name librenms.yourdomain.com;
    root /opt/librenms/html;

    index index.php;
    charset utf-8;

    access_log /var/log/nginx/librenms_access.log;
    error_log /var/log/nginx/librenms_error.log;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
Then save <code> CTRL + O then Enter </code> then exit <code> CTRL + X </code><br/>
Enable site and NGINX:
```
ln -s /etc/nginx/sites-available/librenms /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```
# PHP-FPM Tuning
```bash
nano /etc/php/8.2/fpm/php.ini
```
Copy this in the end of the content
```ini
memory_limit = 512M
upload_max_filesize = 100M
post_max_size = 100M
max_execution_time = 300
date.timezone = Asia/Manila
```
Restart PHP
```
systemctl restart php8.2-fpm
```
You have to switch the PHP version to 8.1 to 8.2 by disabling the php 8.1
```
sudo a2dismod php8.1
sudo systemctl disable php8.1-fpm

sudo systemctl enable php8.2-fpm
sudo systemctl restart php8.2-fpm
```
Switch CLI PHP to 8.2
```
sudo update-alternatives --set php /usr/bin/php8.2
```
Disable Php 8.1
```
sudo a2dismod php8.1
sudo systemctl disable php8.1-fpm
```
Confirm if the version is PHP 8.2.x
```
php -v
```
Reload NGINX and PHP
```
sudo systemctl restart php8.2-fpm
sudo systemctl restart nginx
```
# Update Timezone
To check it
```
timedatectl
```
You must see something like this
```
Time zone: Asia/Manila (PST, +0800)
```
If not lets change the timezone
first in PHP config file
```
sudo nano /etc/php/8.2/fpm/php.ini
```
Then press <code> CTRL + W </code> then type <code> date.timezone </code> then enter<br/>
fill in the ;date.timezone = Asia/Manila then remove the ";" it must look like this.
```
date.timezone = Asia/Manila
```
Then same in CLI Config press <code> CTRL + W </code> then type <code> date.timezone </code> then enter<br/>
```
sudo nano /etc/php/8.2/cli/php.ini
```
fill in the ;date.timezone = Asia/Manila then remove the ";" it must look like this.
```
date.timezone = Asia/Manila
```
Restart PHP & NGINX
```
sudo systemctl restart php8.2-fpm
sudo systemctl restart nginx
```
Then verify if its changed properly
```
php -i | grep "Default timezone"
```
you should see something like this
```java
Default timezone => Asia/Manila
```
# Command Tuning
Run these commands to correct all file ownership and access issues:
```
sudo chown -R librenms:librenms /opt/librenms

sudo setfacl -d -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
sudo chmod -R ug=rwX /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
```
# Input Scheduler
This enables automatic discovery and polling:
```
sudo cp /opt/librenms/dist/librenms-scheduler.service /opt/librenms/dist/librenms-scheduler.timer /etc/systemd/system/
sudo systemctl enable librenms-scheduler.timer
sudo systemctl start librenms-scheduler.timer
```
Set automatic polling
```
sudo cp /opt/librenms/dist/librenms-scheduler.service /opt/librenms/dist/librenms-scheduler.timer /etc/systemd/system/
sudo systemctl daemon-reexec
sudo systemctl enable --now librenms-scheduler.timer
```
Confirm if its Active and waiting
```
systemctl status librenms-scheduler.timer
```
# Add lnms Command
```
sudo ln -s /opt/librenms/lnms /usr/local/bin/lnms
```
# Enable bash Tab completion for lnms
```
sudo cp /opt/librenms/misc/lnms-completion.bash /etc/bash_completion.d/
```
# Restart NGINX
```
sudo systemctl reload nginx
```
# Install Composer
```
cd /opt/librenms
sudo -u librenms ./scripts/composer_wrapper.php install --no-dev
```
# Set Permissions
```
chown -R librenms:librenms /opt/librenms
setfacl -d -m g::rwx /opt/librenms/bootstrap/cache /opt/librenms/storage
setfacl -R -m g::rwx /opt/librenms/bootstrap/cache /opt/librenms/storage
```
# Add Cron Jobs and Logrotate
```
cp /opt/librenms/librenms.nonroot.cron /etc/cron.d/librenms
```
```
cp /opt/librenms/misc/librenms.logrotate /etc/logrotate.d/librenms
```
# Verify if all services is working
```
systemctl enable nginx
systemctl enable php8.1-fpm
systemctl enable mariadb
```
