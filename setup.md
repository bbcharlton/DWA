# Ubunutu 16.04/Wordpress Server Setup
### Using Digital Ocean to spin up an Ubuntu server using MariaDB and PHP for a Wordpress website.
___

> ### 1. Creating Your Droplet

Sign in to your Digital Ocean account and go to the Droplets page. Customize how you want your Droplet to be. Be sure to use **Ubuntu 14.04.5 x64** for this specific setup.

___

> ### 2. New User Setup

It's recommended that you do not use the root user for anything other than creating a new user to work with.

##### Reset Root's Password

After your droplet has been created, click on it. Head over to the **Access** tab and click on the **Reset Root Password** button. An email will be sent to your inbox with a newly generated password. This process may take a while.

##### SSH Into Your Server

<pre>

ssh root@<span style="color: red">your-IP-address</span>

</pre>

You will then be prompted with a notification about the authenticity of the host. Accept the confirmation, then enter the root user's password.


##### Add A New User

<pre>

adduser <span style="color: red">user</span>

</pre>

This will be the user you'll be using instead of root. Type in the necessary information prompted on the screen for your new user.

##### Change The User's Group

<pre>

usermod -aG sudo <span style="color: red">user</span>

</pre>

This gives your newly created user the power to use sudo.

##### Exit And Log Back In

<pre>

exit
ssh <span style="color: red">user</span>@<span style="color: red">your-IP-address</span>

</pre>

Now you will never need to login as root anymore. For now on, you will only use this user account for your server.

___

> ### 3. Nginx Setup

Nginx allows us to be able to use PHP on our server, which Wordpress functionality relies on.

##### Apt-Get And Install Nginx

<pre>

sudo apt-get update
sudo apt-get install nginx

</pre>

This will make sure your server has all the proper files and packages, then install Nginx. This may take a while to finish. Once completed, you should now be able to access your default Nginx page on your server's IP. Do so by visiting **http://<span style="color: red">your-IP-address</span>**.

___

> ### 4. PHP Setup

Nginx alone can't run PHP files, so we'll need to install PHP and configure some files to allow Wordpress to run its files.

##### Apt-Get And Install PHP

<pre>

sudo apt-get install php-fpm php-mysql

</pre>

This installs all the required files and packages to use PHP on your server. This may take a while to complete.

##### Configure The PHP Processor

<pre>

sudo nano /etc/php/7.0/fpm/php.ini

</pre>

Open the php.ini file to access the php-fpm configuration. Use **Ctrl + W** and search for **;cgi.fix_pathinfo=1**. Replace that line with **cgi.fix_pathinfo=0**. Make sure you do not edit any other lines or you'll screw up a lot of stuff!

##### Restart PHP

<pre>

sudo systemctl restart php7.0-fpm.service

</pre>

This makes sure our changes are applied.

##### Configure Nginx To Use The PHP Processor

<pre>

sudo nano /etc/nginx/sites-available/default

</pre>

Open the default file and replace the entire file with the following text:

<pre>

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name localhost;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /var/www/html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}

</pre>

This allows Nginx to read PHP and process the files.

##### Restart Nginx

<pre>

sudo systemctl restart nginx

</pre>

This allows our changes to Nginx using the PHP processer to be applied.

___

> ### 5. Install MariaDB

We will be using MariaDB for our Wordpress database. MariaDB is a version of MySQL with better performance.

##### Apt-Get And Install MariaDB

<pre>

sudo apt-get install mariadb-server
sudo mysql_install_db
sudo service mysql start

</pre>

The installation may take a while to complete. After installation, start the Maria database.

##### Configure MariaDB

<pre>

sudo mysql_secure_installation

</pre>

After running this command, press enter for the default password for MariaDB. Now you will be asked to reset the password again, input **y** for yes and change the password. For the rest of the setup, answer each question with **y** for yes to finish your MariaDB configuration.

##### Login To MariaDB As Root

<pre>

sudo mysql -u root -p

</pre>

Enter the root user's password and you should see a confirmation screen that you are logged in.

##### Create Wordpress Database

<pre>

create database <span style="color: red">database_name</span>;

</pre>

You should then get a success notification if the database is made.

##### Create Wordpress Database User

<pre>

create user <span style="color: red">username</span>@localhost identified by '<span style="color: red">user_password</span>';

</pre>

This creates a new user for the database.

##### Grant New User Privileges

<pre>

grant all privileges on <span style="color: red">database_name</span>.* to <span style="color: red">username</span>@localhost identified by '<span style="color: red">user_password</span>';

</pre>

This grants your new user all privileges for the Wordpress database.

##### Flush Privileges And Exit

<pre>

flush privileges;
exit

</pre>

This makes sure the privilege changes you just made are now active for your new user.

___

> ### 6. Install Wordpress

The final step is to now install Wordpress onto your server, then you should be able to configure your new site on the web!

##### Install Wordpress Files

<pre>

cd /var/www/html
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz

</pre>

Create the necessary directories for your Download and extract the latest Wordpress version onto your server.

##### Wordpress File Configuration Pt. 1

<pre>

cd /var/www/html/wordpress
sudo cp wp-config-sample.php wp-config.php

</pre>

Prepare the Wordpress wp-config.php file to be used.

##### Wordpress File Configuration Pt. 2

<pre>

sudo nano wp-config.php

</pre>

Open the wp-config.php file. Replace **database\_name\_here** with your database name, replace **username_here** with your database user, and replace **password_here** with your user's password. Save and exit the file.

##### Move Wordpress Files

<pre>

sudo rsync -avP /var/www/html/wordpress/ /var/www/html/

</pre>

We will use rsync to move the files along with their privileges and ownership on each file.

##### Give Yourself Ownership

<pre>

sudo chown -R <span style="color: red">user</span>:www-data /var/www/html/*

</pre>

Now you will have ownership over every Wordpress file!

##### Create Uploads Directory

<pre>

sudo mkdir /var/www/html/wp-content/uploads

</pre>

This directory will hold all files you'll ever upload, such as images or favicons.

##### Permit Uploads

<pre>

sudo chown -R :www-data /var/www/html/wp-content/uploads

</pre>

This will allow the web server to create files and directories under this directory, which will permit us to upload content to the server.


##### Restart Nginx And PHP

<pre>

sudo systemctl restart nginx
sudo systemctl restart php7.0-fpm.service

</pre>

This should apply all of our changes and prepare our site to use the Wordpress installation by default.

___

> ### 7. Access And Configure Wordpress

Your Wordpress site should now be available! Visit **http://<span style="color: red">your-IP-address</span>** and the Wordpress site configuration should appear. Now have fun customizing your own Wordpress site!

___






