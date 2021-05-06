# Install Magento 2.4 in Ubuntu 20.04 with NGINX

Install Magento 2.4 in Ubuntu 20.04 with NGINX
Install & Configure NGINX
First of all, update your Ubuntu 20.04 system packages. After that download and install NGINX using following commands in CLI.

sudo apt update
sudo apt install nginx
Download and install NGINX in Ubuntu 20.04
Once the installation is finished, you’ll need to adjust your firewall settings to allow HTTP and HTTPS traffic.
To list all currently available UFW application profiles, you can run:
sudo ufw app list
sudo ufw allow in "Nginx Full"
Check currently available UFW application profiles in ubuntu 20.04
* Nginx Full: This profile opens both port 80 (normal, unencrypted web traffic) and port 443 (TLS/SSL encrypted traffic).
You can enable and verify the change with below command:
sudo ufw enable
sudo ufw status
Enable and verify ufw firewall in ubuntu 20.04
Now open your domain in browser and you will see the default Nginx web page. It should look something like this:
For localhost: http://localhost

Default Nginx Page in ubuntu 20.04
If you see this page, then your web server is now correctly installed and accessible through your firewall.
Verify that the nginx service is active/running and is enabled to automatically start at system startup using the following systemctl commands
sudo systemctl is-active nginx
sudo systemctl is-enabled nginx
sudo systemctl status nginx
Verify that the nginx service status in Ubuntu 20.04
Install MySQL
Install MySQL 8 using apt command
sudo apt install mysql-server
Install MySQL 8 using apt command in Ubuntu 20.04
When prompted, confirm installation by typing Y, and then ENTER
When the installation is finished, it’s recommended that you run a security script that comes pre-installed with MySQL. This script will remove some insecure default settings and lock down access to your database system.
sudo mysql_secure_installation
Answer Y for yes

Select a level of password validation
If you enabled password validation, you’ll be shown the password strength for the root password you just entered and your server will ask if you want to continue with that password. If you are happy with your current password, enter Y for “yes” at the prompt:
For the rest of the questions, press Y and hit the ENTER key at each prompt.
When you’re finished, test if you’re able to log in to the MySQL console by typing:
sudo mysql
This will connect to the MySQL server as the administrative database user root, which is inferred by the use of sudo when running this command. You should see output like this:

To exit the MySQL console, type:
exit
Your MySQL server is now installed and secured. Next, we’ll install PHP.
Install PHP
Install PHP FPM 7.4 and other Packages with below command.
sudo apt install php-fpm php-mysql php7.4-bcmath php7.4-intl php7.4-soap php7.4-curl php-xml php7.4-gd php7.4-zip
Install PHP FPM 7.4 and other Packages on ubuntu
Verify installed PHP version
php -v
Verify installed PHP version in ubuntu 20.04
Add index.php in /etc/nginx/sites-available/default file. Line no: 44

sudo nano /etc/nginx/sites-available/default
Add index.php in nginx default config file in Ubuntu 20.04
Enable the mbstring PHP extension
sudo apt install php7.4-mbstring
sudo phpenmod mbstring
Reload Nginx so the changes take effect:
sudo systemctl reload nginx
Open the php.ini in an editor:
sudo nano /etc/php/7.4/fpm/php.ini
Uncomment the following configuration and change the value to ‘0’. Line no: 798
cgi.fix_pathinfo = 0
Also make changes in memory_limit, max_execution_time and zlib.output_compression as below
memory_limit = 2G
max_execution_time = 1800
zlib.output_compression = On
Open the php.ini files for Command line interface ( CLI ) in an editor:
sudo nano /etc/php/7.4/cli/php.ini
Uncomment the following configuration and change the value to ‘0’. Line no: 798
cgi.fix_pathinfo = 0
Also make changes in memory_limit, max_execution_time and zlib.output_compression as below
memory_limit = 2G
max_execution_time = 1800
zlib.output_compression = On
Verify that the php7.4-fpm service is active/running and is enabled to automatically start at system startup using the following systemctl commands
sudo systemctl start php7.4-fpm
sudo systemctl enable php7.4-fpm
sudo systemctl status php7.4-fpm
Verify that the php7.4 fpm service is active, running, and enabled to automatically start at system startup in ubuntu 20.04
Configure Password Access for the MySQL User
sudo mysql
Or, if you enabled password authentication for the root MySQL user,
mysql -u root -p
SELECT user,authentication_string,plugin,host FROM mysql.user;

Make sure to replace your_secure_password with your password.
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your_secure_password';
SELECT user,authentication_string,plugin,host FROM mysql.user;
exit

Create new MySQL User for Magento 2
Here you can choose more secure name for your user. But for this tutorial I will create magento user.
sudo mysql
Or, if you enabled password authentication for the root MySQL user,
mysql -u root -p
SELECT user,authentication_string,plugin,host FROM mysql.user;
Make sure to replace your_secure_password with your password.
CREATE USER 'magento2'@'localhost' IDENTIFIED BY 'your_secure_password';
ALTER USER 'magento2'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your_secure_password';
GRANT ALL PRIVILEGES ON *.* TO 'magento2'@'localhost' WITH GRANT OPTION;
Create new MySQL User for Magento 2 in Ubuntu
SELECT user,authentication_string,plugin,host FROM mysql.user;

exit
Create Database for Magento 2
mysql -u magento2 -p
CREATE DATABASE magento2;
exit
Install Elastic Search
Install OpenJDK 11
sudo apt install openjdk-11-jdk

Once the installation is complete, verify java version with below command.
java -version
verify java version with below command in Ubuntu
Install Elasticsearch
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
After that, add the Elastic source list to the sources.list.d director
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update

sudo apt install elasticsearch
Install Elastic Search with below command in Ubuntu 20.04
Configure Elasticsearch on Ubuntu 20.04
Create a new file /etc/nginx/conf.d/magento_es_auth.conf with the following contents:
sudo nano /etc/nginx/conf.d/magento_es_auth.conf
server {
   listen 8080;
   location /_cluster/health {
      proxy_pass http://localhost:9200/_cluster/health;
   }
}
sudo service nginx restart
Start the Elasticsearch service with systemctl & enable Elasticsearch to start up every time your server boots.
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
Enable Elasticsearch to start up every time your server boots
Check Elasticsearch proxy is working or not in ubuntu.
curl -i http://localhost:8080/_cluster/health
Check Elasticsearch proxy is working or not in ubuntu.
If should return the output as above.
Install Redis (Optional)
sudo apt install redis-server
Open up the Redis configuration file 
sudo nano /etc/redis/redis.conf
Inside the file, find the supervised directive which allows you to declare an init system and change its value from no to systemd:
Change supervised directive value from no to systemd
why ? because you are running Ubuntu, which uses the systemd init system.
Configure Redis Password
sudo nano /etc/redis/redis.conf
Scroll to the SECURITY section and look for a commented directive requirepass
Remove #, and change foobared to a secure password: ( Line No: 507 )
Remove and change foobared to a secure password
After setting the password, save and close the file, then restart Redis
sudo systemctl restart redis.service
Check Redis is working or not with below command ?
sudo systemctl status redis
Check Redis is working or not
Install Composer 2
Downloading and Installing Composer 2
cd ~
curl -sS https://getcomposer.org/installer -o composer-setup.php
To install composer globally, use the following command which will download and install Composer as a system-wide command named composer, under /usr/local/bin:
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer --version=1.10.16
Check Composer 2 is installed or not in Ubuntu?
composer
Steps to Install Composer on Ubuntu 20.04
Great! Finally we are ready to install Magento 2.
Get your authentication keys
* Login to Magento Marketplace ( https://marketplace.magento.com )
* Go to My Profile > Access Keys Page ( https://marketplace.magento.com/customer/accessKeys/ )
Get Authentication Keys To Install Magento 2.4.1 on Ubuntu 20.04
* Click on Create A New Access Key Button
* Enter a key name and generate
Get the Magento 2.4 metapackage
* Go to root folder and execute below command to change permission.
cd /var/www/html/
sudo chown -R $USER:$USER /var/www/html/
* Create a new Composer project using the Magento Open Source or Magento Commerce metapackage on magento2 directory. If you want you can change the directory name as per your requirement in the below command.
composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition magento2
Install Magento 2.4 with Nginx in Ubuntu 20.04
Set Up Virtual Hosts
Create Virtual Host Config file
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/your_domain
Update /etc/nginx/sites-available/your_domain file as below and make sure to change your_domain with your actual domain name
sudo nano /etc/nginx/sites-available/your_domain
server {
        listen 80;
        listen [::]:80;

        set $MAGE_ROOT /var/www/html/magento2;
        include /var/www/html/magento2/nginx.conf.sample;

        server_name your_domain www.your_domain;

}

upstream fastcgi_backend {
  server  unix:/run/php/php7.4-fpm.sock;
}
Activate Your Virtual Host Configuration file.
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
Reload Nginx so the changes take effect:
sudo systemctl restart nginx
Update Directory & File Permissions for Magento 2
Set read-write permissions so that command line can write files to the Magento 2 file system.
cd /var/www/html/magento2
find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
sudo chown -R :www-data . 
sudo chmod u+x bin/magento
Install Magento 2.4.1
At the time of writing, current version of magento is 2.4.1. So we will install Magento 2.4.1 on Ubuntu 20.04 LTS.
This tutorial assumes that:
* Domain name is http://localhost
* Domain Root Directory is /var/www/html/magento2
* Database Host is on the same machine (localhost)
* Database Name is magento2
* Database User is magento2
* Database Password is admin
So please make sure to update those values as per your requirement.
Go to Domain root Directory:
cd /var/www/html/magento2
Execute below command:
bin/magento setup:install \
--base-url=http://localhost \
--db-host=localhost \
--db-name=magento2 \
--db-user=magento2 \
--db-password=admin \
--admin-firstname=Admin \
--admin-lastname=Admin \
--admin-email=admin@admin.com \
--admin-user=admin \
--admin-password=admin123 \
--language=en_US \
--currency=USD \
--timezone=America/Chicago \
--use-rewrites=1
Once you will execute above command it should successfully install Magento 2.4.1
Install Magento 2.4.1 on Ubuntu 20.04
Setup Google Authenticator for Two Factor Authentication
Below are the steps to Setup 2FA or Fix Two Factor Authentication issue with initial installation of Magento 2.4.1
* Select Google Authenticator as the 2FA provider:
bin/magento config:set twofactorauth/general/force_providers google
* Increase the lifetime of the window to 60 seconds to prevent tokens from expiring.
bin/magento config:set twofactorauth/google/otp_window 60
* Generate a Base32-encoded string for the shared secret value
For example, encoding the string secretkey1 with the online [Base32 Encode][13] tool ( https://cryptii.com/pipes/base32 ) returns the value ONSWG4TFORVWK6JR.
Please make sure to change ONSWG4TFORVWK6JR string as per your secret key in below commands
* Add the encoded shared secret to Google Authenticator.
bin/magento security:tfa:google:set-secret admin ONSWG4TFORVWK6JR
Setup Google Authenticator for Two Factor Authentication in Magento 2 1
* Open Google Authenticator Application on your mobile and Add new Key ONSWG4TFORVWK6JR
* Once you will add Your key you will see Authenticator code in application. Copy that Authenticator code and use it to login in magento 2 admin panel.
two factor authentication in magento 2.4Magento 2 Admin PanelMagento 2 Front End
Disable Magento_TwoFactorAuth Module in Magento 2.4
Two Factor Authentication issue in Magento 2.4.1
If you are installing magento2 on local system or want a quick access you can execute below command to disable Magento_TwoFactorAuth module in magento 2.4.1
sudo bin/magento module:disable Magento_TwoFactorAuth
Disable Two Factor Authentication in Magento 2.4
If you want to enable Magento_TwoFactorAuth module you can use below command
sudo bin/magento module:enable Magento_TwoFactorAuth
I hope you are able to install Magento 2.4 with this guide. Please let me know in the comment below if you are facing any issue in installation with this guide.

Copy: https://techstorm.io/install-magento-2-in-ubuntu-20-04-with-nginx/

