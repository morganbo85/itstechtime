# Step 1
### Create a server running Ubuntu 20.04
`Create Linode`

### First Login run updates
To ensure that all the installed packages are up to date and that PHP is available in the APT repository run the below commands.
`sudo apt update && sudo apt upgrade -y`

### Then Restart.
`sudo apt reboot`

# Step 2 Set a Domain (Optional if self Hosting)

### Purchase a domain

### Make an A record pointing to your IP address

### Set Your Domain Name

```
my_domain="Your.Domain.tld"
echo $my_domain

hostnamectl set-hostname $my_domain
hostname -f
```

# Step 3 Create the occ Helper Script

Create a helper script to simplify running occ commands:

```
FILE="/usr/local/bin/occ"
cat <<EOM >$FILE
#! /bin/bash
cd /var/www/owncloud
sudo -E -u www-data /usr/bin/php /var/www/owncloud/occ "\$@"
EOM
```

Make the helper script executable:

```
chmod +x $FILE
```

# Step 4 Install the Required Packages

```
apt install -y \
  apache2 libapache2-mod-php \
  mariadb-server openssl redis-server wget php-imagick \
  php-common php-curl php-gd php-gmp php-bcmath php-imap \
  php-intl php-json php-mbstring php-mysql php-ssh2 php-xml \
  php-zip php-apcu php-redis php-ldap php-phpseclib
```

# Step 5 Install smbclient php Module (Optional)

If you want to connect to external storage via SMB, install the smbclient php module.

First, install the required packages:

```
apt-get install -y libsmbclient-dev php-dev php-pear
```
Then install smblclient php module using pecl:

```
pecl channel-update pecl.php.net
mkdir -p /tmp/pear/cache
pecl install smbclient-stable
echo "extension=smbclient.so" > /etc/php/7.4/mods-available/smbclient.ini
phpenmod smbclient
systemctl restart apache2
```

Check if it was successfully activated:

```
php -m | grep smbclient
```

This should show the following output:

```
libsmbclient
smbclient
```

# Step6 Install the Recommended Packages

Additional valuable tools helpful for debugging:

```
apt install -y \
  unzip bzip2 rsync curl jq \
  inetutils-ping  ldap-utils\
  smbclient
```

# Step8 Configure Apache
#### Create a Virtual Host Configuration

```
FILE="/etc/apache2/sites-available/owncloud.conf"
cat <<EOM >$FILE
<VirtualHost *:80>
# uncommment the line below if variable was set
#ServerName $my_domain
DirectoryIndex index.php index.html
DocumentRoot /var/www/owncloud
<Directory /var/www/owncloud>
  Options +FollowSymlinks -Indexes
  AllowOverride All
  Require all granted

 <IfModule mod_dav.c>
  Dav off
 </IfModule>

 SetEnv HOME /var/www/owncloud
 SetEnv HTTP_HOME /var/www/owncloud
</Directory>
</VirtualHost>
EOM
```

#### Enable the Virtual Host Configuration

```
a2dissite 000-default
a2ensite owncloud.conf
```
# Step7 Configure the Database

|   |   |
|---|---|
||I recommended setting a strong password for the database user.|

Ensure transaction-isolation level is set and performance_schema on.

```
sed -i "/\[mysqld\]/atransaction-isolation = READ-COMMITTED\nperformance_schema = on" /etc/mysql/mariadb.conf.d/50-server.cnf
systemctl start mariadb
mysql -u root -e \
  "CREATE DATABASE IF NOT EXISTS owncloud; \
  CREATE USER IF NOT EXISTS 'owncloud'@'localhost' IDENTIFIED BY 'password'; \
  GRANT ALL PRIVILEGES ON *.* TO 'owncloud'@'localhost' WITH GRANT OPTION; \
  FLUSH PRIVILEGES;"
```

# Step8 Enable the Recommended Apache Modules

```
a2enmod dir env headers mime rewrite setenvif
systemctl restart apache2
```

# Step9 Download ownCloud
```
cd /var/www/
wget https://download.owncloud.com/server/stable/owncloud-complete-latest.tar.bz2 && \
tar -xjf owncloud-complete-latest.tar.bz2 && \
chown -R www-data. owncloud
```

# Step10 Install ownCloud

|   |   |
|---|---|
||Set a strong password for your owncloud admin user and the database user.|

```
occ maintenance:install \
    --database "mysql" \
    --database-name "owncloud" \
    --database-user "owncloud" \
    --database-pass "password" \
    --data-dir "/var/www/owncloud/data" \
    --admin-user "admin" \
    --admin-pass "admin"
```

# Step11 Configure ownCloud’s Trusted Domains

```
my_ip=$(hostname -I|cut -f1 -d ' ')
occ config:system:set trusted_domains 1 --value="$my_ip"
occ config:system:set trusted_domains 2 --value="$my_domain"
```

# Step12 Configure the cron Jobs

Set your background job mode to cron:

```
occ background:cron
```
Configure the execution of the cron job to every 15 min and the cleanup of chunks every night at 2 am:

```
echo "*/15  *  *  *  * /var/www/owncloud/occ system:cron" \
  | sudo -u www-data -g crontab tee -a \
  /var/spool/cron/crontabs/www-data
echo "0  2  *  *  * /var/www/owncloud/occ dav:cleanup-chunks" \
  | sudo -u www-data -g crontab tee -a \
  /var/spool/cron/crontabs/www-data
```

|   |   |
|---|---|
||If you need to sync your users from an LDAP or Active Directory Server, add this additional [Cron job](https://doc.owncloud.com/server/next/admin_manual/configuration/server/background_jobs_configuration.html). Every 4 hours this cron job will sync LDAP users in ownCloud and disable the ones who are not available for ownCloud. Additionally, you get a log file in `/var/log/ldap-sync/user-sync.log` for debugging.|

```
echo "1 */6 * * * /var/www/owncloud/occ user:sync \
  'OCA\User_LDAP\User_Proxy' -m disable -vvv >> \
  /var/log/ldap-sync/user-sync.log 2>&1" \
  | sudo -u www-data -g crontab tee -a \
  /var/spool/cron/crontabs/www-data
mkdir -p /var/log/ldap-sync
touch /var/log/ldap-sync/user-sync.log
chown www-data. /var/log/ldap-sync/user-sync.log
```

# Step13 Configure Caching and File Locking

```
occ config:system:set \
   memcache.local \
   --value '\OC\Memcache\APCu'
occ config:system:set \
   memcache.locking \
   --value '\OC\Memcache\Redis'
occ config:system:set \
   redis \
   --value '{"host": "127.0.0.1", "port": "6379"}' \
   --type json
```
### Configure Log Rotation

```
FILE="/etc/logrotate.d/owncloud"
sudo cat <<EOM >$FILE
/var/www/owncloud/data/owncloud.log {
  size 10M
  rotate 12
  copytruncate
  missingok
  compress
  compresscmd /bin/gzip
}
EOM
```

# Step14 Finalize the Installation

Make sure the permissions are correct:

```
cd /var/www/
chown -R www-data. owncloud
```

**ownCloud is now installed. You can confirm by pointing your web browser to your ownCloud installation.**

To check if you have installed the correct version of ownCloud and that the occ command is working, execute the following:

```
occ -V
echo "Your ownCloud is accessable under: "$my_ip
echo "Your ownCloud is accessable under: "$my_domain
echo "The Installation is complete."
```
Source: https://doc.owncloud.com/server/next/admin_manual/installation/quick_guides/ubuntu_20_04.html

# Step15 Enable HTTPS (Optional)
### Make dir to hold self signed ssl
`sudo mkdir /etc/apache2/ssl`
### Create self signed certificates
`openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/owncloud.key -out /etc/apache2/ssl/owncloud.crt`

### Edit default-ssl.conf
`sudo nano /etc/apache2/sites-available/default-ssl.conf`

Add Server name at the top and change Document Root
```
 ServerName 192.168.10.112:443
 DocumentRoot /var/www/owncloud
```

Point SSLCert to the right directory
```
SSLCertificateFile      /etc/apache2/ssl/owncloud.crt
SSLCertificateKeyFile /etc/apache2/ssl/owncloud.key
```

### Enable default-ssl.conf
`a2ensite default-ssl.conf`
### Enable SSL model
`a2enmod ssl`
### Edit Owncloud.conf to redirect
```
sudo nano /etc/apache2/sites-available/owncloud.conf

<VirtualHost *:80>
# uncommment the line below if variable was set
#ServerName
redirect "/" https://172.233.212.71/ # IP will be different for your machine
DirectoryIndex index.php index.html

```
### Restart Apache
`sudo service apache2 restart`
### Verify by visiting IP address
`https://172.233.212.71`
# Resetting a Lost Admin Password

The normal ways to recover a lost password are:

1. Click the password reset link on the login screen; this appears after a failed login attempt. This works only if you have entered your email address on your Personal page in the ownCloud Web interface, so that the ownCloud server can email a reset link to you.
    
2. Ask another ownCloud server admin to reset it for you.
    

If neither of these is an option, then you have a third option, and that is using the `occ` command. `occ` is in the `owncloud` directory, for example `/var/www/owncloud/occ`. `occ` has a command for resetting all user passwords, `user:resetpassword`. It is best to run `occ` as the HTTP user, as in this example on Ubuntu Linux:

```
$ sudo -u www-data ./occ user:resetpassword admin
Enter a new password:
Confirm the new password:
Successfully reset password for admin
```

If your ownCloud username is not `admin`, then substitute your ownCloud username.

You can find your HTTP user in your HTTP configuration file. These are the default Apache HTTP user: group on Linux distros:

- Centos, Red Hat, Fedora: `apache:apache`
    
- Debian, Ubuntu, Linux Mint: `www-data:www-data`
    
- openSUSE: `wwwrun:www`
