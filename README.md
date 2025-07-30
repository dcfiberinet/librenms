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
<div><p color="red">Note : </p><p>Replace the <code>YourStrongPassword</code> with your preferred Secured Password this will be used upon setting up the webui of LibreNMS</p>

