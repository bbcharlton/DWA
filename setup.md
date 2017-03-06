# Ubunutu 16.04/Wordpress Ansible Automation
### Using Digital Ocean to spin up an Ubuntu server using MariaDB and PHP for a Wordpress website through Ansible's automated playbooks.
___

> ### 1. Generate Your SSH Key

##### Create SSH Key

<pre>

cd ~/.ssh
ssh-keygen -t rsa -b 4096 -C "<span style="color: red">your@emailaddress</span>" -f id_<span style="color: red">id-name

</pre>

On your local machine, change to your ssh directory and generate a new ssh key to use for your Digital Ocean server. It will then generate and ask you for a passphrase to remember. Give it a secure password and press enter. Get your public key using the following command:

<pre>

cat id_<span style="color: red">id-name</span>.pub

</pre>

This outputs our public ssh key. Copy this key. We will need it for our Digital Ocean steps.

___

> ### 2. Creating Your Droplet With SSH

Sign in to your Digital Ocean account and go to your settings page. Click on **Security** and click the **Add SSH Key** button. Paste in your copied public key and give it a unique name. Save and go to the Droplets page. Customize how you want your Droplet to be. Be sure to use **Ubuntu 16.04 x64** for this specific setup. For the SSH option, click on the SSH key name you just added to your profile, and create

___

> ### 3. Install Ansible With Brew

##### Install Brew

You can use the following article on how to install [Brew](https://brew.sh/).

##### Install Ansible

<pre>

brew install ansible

</pre>

This installs Ansible globally on our local machine, which allows us to use it for any local project.

___

> ### 4. Setup Anstible Directory

##### Prepare Ansible Directory

<pre>

mkdir <span style="color: red">directory-name</span>
cd <span style="color: red">directory-name</span>
nano hosts

</pre>

This creates our Ansible directory. The host file will contain all of our server IPs listed in groups. Create a group by typing **[<span style="color: red">group-name</span>]** then list your server IPs below it. Save and exit the file.

##### Install Python

<pre>

ansible all -m raw -s -a "sudo apt-get -y install python-simplejson" -u root --private-key=~/.ssh/id_<span style="color: red">id-name</span> -i ./hosts

</pre>

We are required to use the raw module in order to get Python working on our Ansible automation. This will allow us to use Python for future steps. This may take a while.

##### Ping Our Server

<pre>

ansible all -m ping -u root --private-key=~/.ssh/id_<span style="color: red">id-name</span> -i ./hosts

</pre>

You will be asked about the authenticity of the host. Enter **y** for yes to add the IP to your known hosts. We should then successfully ping the server.

___

> ### 4. Setup Anstible Files

##### Prepare Ansible Files

<pre>

mkdir roles
cd roles
mkdir server
cd server
mkdir files
mkdir handlers
mkdir meta
mkdir tasks
mkdir templates
mkdir vars

</pre>

This creates the necessary file structure for Ansible to work. Add a **main.yml** file to the following folders:

* **files**
* **handlers**
* **meta**
* **tasks**
* **vars**

The **main.yml** files will hold all of our functionality for Ansible's automation.

___

> ### 5. Main.yml Files

##### Files Folder

In your **files** folder, visit the [H5BP Github Repo](https://github.com/h5bp/server-configs-nginx) and download the H5BP folder. Move it into your **files** folder. This will allow for more nginx config settings.

##### Handlers Folder

In your **handlers** folder, paste the following text into its **main.yml** file:

<pre>

---
- name: Start Nginx
  service: name=nginx state=started

- name: Reload Nginx
  service: name=nginx state=restarted

- name: Stop Nginx
  service: name=nginx state=stopped

- name: Reload PHP
  service: name=php7.0-fpm state=restarted

</pre>

This will allow us to start, restart, and stop Nginx, as well as restart PHP. Our tasks will eventually call these handlers after certain events are fired.

##### Meta Folder

In your **meta** folder, paste the following text into its **main.yml** file:

<pre>

---
dependencies: []

</pre>

This allows the meta to store any dependencies.

##### Tasks Folder

In your **tasks** folder, paste the following text into its **main.yml** file:

<pre>

---
- script: ./git-setup.sh

- user:
    name: bailey
    group: sudo

- name: Add Nginx Repo
  apt_repository: repo='ppa:nginx/stable' state=present

- name: Install Nginx
  apt: pkg=nginx state=latest update_cache=true
  notify:
    - Start Nginx

- name: Add PHP Repo
  apt_repository: repo='ppa:ondrej/php' state=present

- name: Install PHP
  apt: pkg={{ item }} state=latest update_cache=true
  with_items:
    - php7.0-fpm
    - php7.0-mysql

- name: Install MariaDB repository
  apt_repository: repo='deb http://ftp.igh.cnrs.fr/pub/mariadb/repo/10.0/ubuntu trusty main' state=present 

- name: Add repository key to the system 
  apt_key: keyserver=keyserver.ubuntu.com id=0xcbcb082a1bb943db 

- name: Install MariaDB Server 
  apt: pkg={{ item }} state=latest update_cache=true
  with_items:
    - mariadb-server
    - python-mysqldb

- name: Download WordPress
  get_url: url=http://wordpress.org/wordpress-{{ wp_version }}.tar.gz dest=/var/www/html/wordpress-{{ wp_version }}.tar.gz

- name: Extract Wordpress Files
  command: chdir=/var/www/html /bin/tar xvf wordpress-{{ wp_version }}.tar.gz creates=/var/www/html/wordpress

- name: Add group "wordpress"
  group: name=wordpress

- name: Add user "wordpress"
  user: name=wordpress group=wordpress home=/var/www/html/wordpress

- name: Create WordPress database
  mysql_db: name={{ wp_db_name }} state=present

- name: Create WordPress database user
  mysql_user: name={{ wp_db_user }} password={{ wp_db_password }} priv={{ wp_db_name }}.*:ALL host='localhost' state=present

- name: Copy WordPress config file
  template: src=wp-config.php dest=/var/www/html/wordpress

- name: Change ownership of WordPress installation
  file: path=/var/www/html/wordpress owner=wordpress group=wordpress state=directory recurse=yes setype=httpd_sys_content_t

- name: Add H5BP Config
  copy: src=h5BP dest=/etc/nginx owner=root group=root

- name: Remove Default Config
  file: dest=/etc/nginx/sites-enabled/default state=absent
  notify:
    - Reload Nginx

- name: Add Server {{ domain }} Config
  template: src={{ domain }}.j2 dest=/etc/nginx/sites-available/{{ domain }} owner=root group=root

- name: Enable Site Config
  file: src=/etc/nginx/sites-available/{{ domain }} dest=/etc/nginx/sites-enabled/{{ domain }} state=link
  notify:
    - Reload Nginx
    - Reload PHP

</pre>

This is the core functionality for our Ansible playbook we'll be creating soon. This installs and runs Nginx, PHP, MariaDB, and Wordpress, as well as setting up our configuration for our site.

##### Template Folder

In your **template** folder, create a new file. Give it a domain name, for example: **poop123.com** and add the extension **.j2**. So your final file name should be something like **poop123.com.j2**. Inside of that file, paste the following text:

<pre>

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html/wordpress;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name localhost;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /var/www/html/wordpress;
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

This will be our Nginx root document that points to specific locations for our Wordpress files and PHP to be ran. Next, you'll need to create a **wp-config.php** file and paste the following text:

```shell

<?php
/**
 * The base configurations of the WordPress.
 *
 * This file has the following configurations: MySQL settings, Table Prefix,
 * Secret Keys, WordPress Language, and ABSPATH. You can find more information
 * by visiting {@link http://codex.wordpress.org/Editing_wp-config.php Editing
 * wp-config.php} Codex page. You can get the MySQL settings from your web host.
 *
 * This file is used by the wp-config.php creation script during the
 * installation. You don't have to use the web site, you can just copy this file
 * to "wp-config.php" and fill in the values.
 *
 * @package WordPress
 */
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', '{{ wp_db_name }}');
/** MySQL database username */
define('DB_USER', '{{ wp_db_user }}');
/** MySQL database password */
define('DB_PASSWORD', '{{ wp_db_password }}');
/** MySQL hostname */
define('DB_HOST', 'localhost');
/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8');
/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');
/**#@+
 * Authentication Unique Keys and Salts.
 *
 * Change these to different unique phrases!
 * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
 *
 * @since 2.6.0
 */

/**#@-*/
/**
 * WordPress Database Table prefix.
 *
 * You can have multiple installations in one database if you give each a unique
 * prefix. Only numbers, letters, and underscores please!
 */
$table_prefix  = 'wp_';
/**
 * WordPress Localized Language, defaults to English.
 *
 * Change this to localize WordPress. A corresponding MO file for the chosen
 * language must be installed to wp-content/languages. For example, install
 * de_DE.mo to wp-content/languages and set WPLANG to 'de_DE' to enable German
 * language support.
 */
define('WPLANG', '');
/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 */
define('WP_DEBUG', false);
/** Disable Automatic Updates Completely */
define( 'AUTOMATIC_UPDATER_DISABLED', {{auto_up_disable}} );
/** Define AUTOMATIC Updates for Components. */
define( 'WP_AUTO_UPDATE_CORE', {{core_update_level}} );
/* That's all, stop editing! Happy blogging. */
/** Absolute path to the WordPress directory. */
if ( !defined('ABSPATH') )
	define('ABSPATH', dirname(__FILE__) . '/');
/** Sets up WordPress vars and included files. */
require_once(ABSPATH . 'wp-settings.php');


```

This is our Wordpress' configuration file that includes our database information.

##### Vars Folder

In your **vars** folder, paste the following text into its **main.yml** file:

<pre>

---
domain: <span style="color: red;">domain@name.com</span>
wp_version: <span style="color: red;">4.7.2</span>
wp_db_name: <span style="color: red;">wordpressdb</span>
wp_db_user: <span style="color: red;">wpuser</span>
wp_db_password: <span style="color: red;">dbpassword</span>
auto_up_disable: false
core_update_level: true

</pre>

These are all of the variables that will be used throughout our Ansible playbook. You can change the Wordpress version, but **4.7.2** is recommended.

___

> ### 6. Playbook File

##### Create Playbook.yml

In the root of your Ansible project, create a **playbook.yml** file and add the following text:

<pre>

---
  - hosts: <span style="color: red;">group name</span>
    become: true
    user: root
    roles:
      - server

</pre>

This will actually execute our **server** role to run on every group's IPs in our **hosts** file.

___

> ### 7. Githook Script

##### Create Bare Repo For Githook Setup

In the root of your Ansible project, create a **git-setup.sh** file and add the following text:

```shell

#!/bin/bash

cd /var

if [ ! -d /var/repos ]; then
  mkdir repos && cd repos
  mkdir wp.git && cd wp.git
  git init --bare
  cd hooks
  touch post-receive
  chmod +x post-receive
  echo "#!/bin/bash" > post-receive
  echo "git --work-tree=/var/www/html/wordpress/ --git-dir=/var/repos/wp.git/ checkout -f" >> post-receive
fi


```

This will check if your repo already exists, if not, it will create the new repo and your githook for post-receive.

___

> ### 8. Ansible Automation

##### Run Ansible Playbook

<pre>

ansible-playbook --private-key=~/.ssh/id_<span style="color: red">id-name</span> -i ./hosts playbook.yml

</pre>

This will run our playbook and automate installing everything we need onto our servers. This may take a while to finish running, but you can watch the progress as it runs.

___

> ### 9. Access And Configure Wordpress

Your Wordpress site should now be available! Visit **http://<span style="color: red">your-IP-address</span>** and the Wordpress site configuration should appear. Now have fun customizing your own Wordpress site!

___






