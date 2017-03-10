# Ubunutu 14.04/Wordpress Server Setup
### Using Digital Ocean to spin up an Ubuntu server using MariaDB and PHP for a Wordpress website.
___

> ### 1. Generate Your SSH Key

##### Create SSH Key

##### Note: If you already have your own ssh key, you can use that key instead of generating a new one.

```shell
cd ~/.ssh
ssh-keygen -t rsa -b 4096 -C "YOUR@EMAIL_ADDRESS" -f id_ID_NAME
```

On your local machine, change to your ssh directory and generate a new ssh key to use for your Digital Ocean server. It will then generate and ask you for a passphrase to remember. Give it a secure password and press enter. Get your public key using the following command:

```shell
cat id_ID_NAME.pub
```

This outputs our public ssh key. Copy this key. We will need it for our Digital Ocean steps.

___

> ### 2. Creating Your Droplet With SSH

Sign in to your Digital Ocean account and go to your settings page. Click on **Security** and click the **Add SSH Key** button. Paste in your copied public key and give it a unique name. Save and go to the Droplets page. Customize how you want your Droplet to be. Be sure to use **Ubuntu 14.04 x64** for this specific setup. For the SSH option, click on the SSH key name you just added to your profile, and create

```shell
ssh root@YOUR_IP_ADDRESS
```

Now you can enter into your server securely without a password. You will be asked about the authenticity of the host. Enter **y** for yes to add the IP to your known hosts. If you have problems ssh'ing into your server, follow this [guide](https://www.digitalocean.com/community/questions/permission-denied-after-creating-droplet-using-ssh-keys).

___

> ### 3. Nginx Setup

Nginx allows us to be able to use PHP on our server, which Wordpress functionality relies on.

##### Apt-Get And Install Nginx

```shell
apt-get update
apt-get install nginx
```

This will make sure your server has all the proper files and packages, then install Nginx. This may take a while to finish. Once completed, you should now be able to access your default Nginx page on your server's IP. Do so by visiting **http://YOUR\_IP\_ADDRESS**.

___

> ### 4. PHP Setup

Nginx alone can't run PHP files, so we'll need to install PHP and configure some files to allow Wordpress to run its files.

##### Apt-Get And Install PHP

```shell
apt-get install php5-fpm php5-mysql
```

This installs all the required files and packages to use PHP on your server. This may take a while to complete.

##### Configure The PHP Processor

```shell
nano /etc/php5/fpm/php.ini
```

Open the php.ini file to access the php-fpm configuration. Use **Ctrl + W** and search for **;cgi.fix_pathinfo=1**. Replace that line with the following:

```shell
cgi.fix_pathinfo=0
``` 

##### Restart PHP

```shell
service php5-fpm restart
```

This makes sure our changes are applied.

##### Configure Nginx To Use The PHP Processor

```shell
cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default-backup
rm /etc/nginx/sites-available/default
nano /etc/nginx/sites-available/default
```

We will first make a backup of the current server document, delete the original, and create a new one. Paste the following text:

```shell
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /usr/share/nginx/html;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name localhost;

    location / {
        try_files $uri $uri/ =404;
    }
    
    location /favicon.ico {
        log_not_found off; 
        access_log off;
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

This allows Nginx to read PHP and process the files.

##### Restart Nginx

```shell
service nginx restart
```

This allows our changes to Nginx using the PHP processer to be applied.

___

> ### 5. Install MariaDB

We will be using MariaDB for our Wordpress database. MariaDB is a version of MySQL with better performance.

##### Apt-Get And Install MariaDB

```shell
apt-get install mariadb-server
```

You will then be prompted to reset the root user's password. The installation may take a while to complete.

##### Configure MariaDB

```shell
mysql_secure_installation
```

This command ensures that our database is secured by removing all default databases and anonynmous users. After running this command, press enter for the default password for MariaDB. Now you will be asked to reset the password again, input **n** for no. For the rest of the setup, answer each question with **y** for yes to finish your MariaDB configuration.

##### Login To MariaDB As Root

```shell
mysql -u root -p
```

Enter the root user's password and you should see a confirmation screen that you are logged in.

##### Create Wordpress Database

```shell
create database DATABASE_NAME;
```

You should then get a success notification if the database is made.

##### Create Wordpress Database User

```shell
create user USERNAME@localhost identified by 'USERNAME_PASSWORD';
```

##### Grant New User Privileges

```shell
grant all privileges on DATABASE_NAME.* to USERNAME@localhost identified by 'USERNAME_PASSWORD';
```

This grants your new user all privileges for the Wordpress database.

##### Flush Privileges And Exit

```shell
flush privileges;
exit
```

This makes sure the privilege changes you just made are now active for your new user.

___

> ### 6. Install Wordpress

The final step is to now install Wordpress onto your server, then you should be able to configure your new site on the web!

##### Install Wordpress Files

```shell
cd /usr/share/nginx/html
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
```

Download and extract the latest Wordpress version onto your server.

##### Wordpress File Configuration Pt. 1

```shell
cd wordpress
cp wp-config-sample.php wp-config.php
```

Prepare the Wordpress wp-config.php file to be used.

##### Wordpress File Configuration Pt. 2

```shell
nano wp-config.php
```

Open the wp-config.php file. Replace **database\_name\_here** with your database name, replace **username_here** with your database user, and replace **password_here** with your user's password. Save and exit the file.

##### Move Wordpress Files

```shell
rsync -avP /usr/share/nginx/html/wordpress/ /usr/share/nginx/html/
```

We will use rsync to move the files along with their privileges and ownership on each file.

##### Create Uploads Directory

```shell
mkdir /usr/share/nginx/html/wp-content/uploads
```

This directory will hold all files you'll ever upload.

##### Restart Nginx And PHP

```shell
service nginx restart
service php5-fpm restart
```

This should apply all of our changes and prepare our site to use the Wordpress installation by default.

___

> ### 7. Access And Configure Wordpress

Your Wordpress site should now be available! Visit **http://YOUR\_IP\_ADDRESS** and the Wordpress site configuration should appear. Now have fun customizing your own Wordpress site!

___






