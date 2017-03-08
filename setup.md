# Ubunutu 16.04/Pipeline Ansible Automation
### Using Digital Ocean to spin up an Ubuntu server that uses three different pipelines, HTML, Wordpress, and Node.
___

> ### 1. Generate Your SSH Key

##### Create SSH Key

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

Sign in to your Digital Ocean account and go to your settings page. Click on **Security** and click the **Add SSH Key** button. Paste in your copied public key and give it a unique name. Save and go to the Droplets page. Customize how you want your Droplet to be. Be sure to use **Ubuntu 16.04 x64** for this specific setup. For the SSH option, click on the SSH key name you just added to your profile, and create

___

> ### 3. Install Ansible With Brew

##### Install Brew

You can use the following article on how to install [Brew](https://brew.sh/).

##### Install Ansible

```shell
brew install ansible
```

This installs Ansible globally on our local machine, which allows us to use it for any local project.

___

> ### 4. Setup Anstible Directory

##### Prepare Ansible Directory

```shell
mkdir DIRECTORY_NAME
cd DIRECTORY_NAME
nano hosts
```

This creates our Ansible directory. The host file will contain all of our server IPs listed in groups. Create a group by typing **[GROUP_NAME]** then list your server IP below it. Save and exit the file.

#### Warning: For creating a Staging and Production server, it is suggested to only use one IP at a time for the rest of this tutorial so both servers aren't running on one IP, since Ansible automates the same server document onto all servers.

##### Install Python

```shell
ansible all -m raw -s -a "sudo apt-get -y install python-simplejson" -u root --private-key=~/.ssh/id_ID_NAME -i ./hosts
```

We are required to use the raw module in order to get Python working on our Ansible automation. This will allow us to use Python for future steps. This may take a while. You will be asked about the authenticity of the host. Enter **y** for yes to add the IP to your known hosts.

##### Ping Our Server

```shell
ansible all -m ping -u root --private-key=~/.ssh/id_ID_NAME -i ./hosts
```

We should now get two successful outputs from our server pings.

___

> ### 4. Setup Ansible Files

##### Prepare Ansible Files

```shell
mkdir roles
cd roles
mkdir nginx
mkdir wordpress
mkdir node
mkdir html
mkdir githook
```

These will be our roles running separate tasks. This creates the necessary file structure for Ansible to work. For each role, add the following folders:

* **files**
* **handlers**
* **meta**
* **tasks**
* **templates**
* **vars**

Then add a **main.yml** file to the following folders for each role:

* **handlers**
* **meta**
* **tasks**
* **vars**

The **main.yml** files will hold all of our functionality for Ansible's automation.

___

> ### 5. Nginx Role

##### Handlers Folder

For your **nginx** role, in the **handlers** folder, paste the following text into its **main.yml** file:

```shell
---
- name: Start Nginx
  service: name=nginx state=started

- name: Reload Nginx
  service: name=nginx state=restarted

- name: Stop Nginx
  service: name=nginx state=stopped
```

This will allow us to start, restart, and stop Nginx. Our tasks will eventually call these handlers after certain events are fired.

##### Meta Folder

For your **nginx** role, in the **meta** folder, paste the following text into its **main.yml** file:

```shell
---
dependencies: []
```

This allows the meta to store any dependencies.

##### Tasks Folder

For your **nginx** role, in the **tasks** folder, paste the following text into its **main.yml** file:

```shell
---
- script: ./directory-setup.sh

- name: Add Nginx Repo
  apt_repository: repo='ppa:nginx/stable' state=present

- name: Install Nginx
  apt: pkg=nginx state=latest update_cache=true
  notify:
    - Start Nginx

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
```

Tasks are the core functionality for Ansible's playbooks. This task will get Nginx installed and running.

##### Templates Folder

For your **nginx** role, in the **templates** folder, create a new file. Give it a domain name, for example: **poop123.com** and add the extension **.j2**. So your final file name should be something like **poop123.com.j2**. Inside of that file, paste the following text:

```shell
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html/html;
    index index.html index.htm index.nginx-debian.html;

    server_name html.{{ ip }}.xip.io;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /var/www/html;
    }
}

server {
    listen 80;
    listen [::]:80;

    root /var/www/html/wordpress;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name php.{{ ip }}.xip.io;

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

server {
    listen 80;
    listen [::]:80;

    root /var/www/html/node;
    index index.html index.htm index.nginx-debian.html;

    server_name node.{{ ip }}.xip.io;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
    }
}
```

##### Note: We will need to update this document's domain IP used later because automation will set both servers to one IP.

This will be our Nginx root document that points to specific locations for our pipelines to work.

##### Vars Folder

For your **nginx** role, in the **vars** folder, paste the following text into its **main.yml** file:

```shell
---
ip: YOUR_IP_ADDRESS
domain: YOUR@DOMAIN_NAME
```

This domain variable should be the same as your **.j2** file's domain name.

___

> ### 6. HTML Role

##### Files Folder

For your **html** role, in the **files** folder, create a new file named **index.html** and paste the following text:

```shell
<h1>Hello, world!</h1>
```

##### Meta Folder

For your **html** role, in the **meta** folder, paste the following text into its **main.yml** file:

```shell
---
dependencies: []
```

##### Tasks Folder

For your **html** role, in the **tasks** folder, paste the following text into its **main.yml** file:

```shell
---
- file:
    path=/var/www/html/html
    recurse=yes
    state=directory

- name: Add index.html file  copy: src=index.html dest=/var/www/html/html/index.html owner=root group=root
```

___

> ### 7. Wordpress Role

##### Handlers Folder

For your **wordpress** role, in the **handlers** folder, paste the following text into its **main.yml** file:

```shell
---
- name: Reload PHP
  service: name=php7.0-fpm state=restarted
```

This will allow us to reload our PHP.

##### Meta Folder

For your **wordpress** role, in the **meta** folder, paste the following text into its **main.yml** file:

```shell
---
dependencies: []
```

##### Tasks Folder

For your **wordpress** role, in the **tasks** folder, paste the following text into its **main.yml** file:

```shell
---
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
  get_url: url=http://wordpress.org/wordpress-{{ wp_version }} dest=/var/www/html/wordpress-{{ wp_version }}.tar.gz

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
```

This will do all the installation for our Wordpress site and Maria database.

##### Templates Folder

For your **wordpress** role, in the **templates** folder, create a new file named **wp-config.php** and paste the following text:

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

This is the config file we will be adding in to use our new Maria database.

##### Vars Folder

For your **wordpress** role, in the **vars** folder, paste the following text into its **main.yml** file:

```shell
---
wp_version: 4.7.2
wp_db_name: wordpressdb
wp_db_user: wpuser
wp_db_password: DATABASE_PASSWORD
auto_up_disable: false
core_update_level: true
```

This will be the information for our Maria database. Don't forget to change the password.

___

> ### 8. Node Role

##### Files Folder

For your **node** role, in the **files** folder, create a new file named **server.js** and paste the following text:

```shell
var http = require('http');

var server = http.createServer(function (request, response) {
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.end("Hello, World!");
});

server.listen(3000);

console.log('Running Node app on port 3000.');
```

Also create a new file named **install-node.sh** and paste the following text:

```shell
#!/bin/bash
cd ~
curl -sL https://deb.nodesource.com/setup_6.x -o nodesource_setup.sh
bash nodesource_setup.sh
apt-get install nodejs
apt-get install build-essential -y
ufw allow 'Nginx Full'
npm install -g pm2
cd /var/www/html/node
pm2 start server.js
```

##### Meta Folder

For your **node** role, in the **meta** folder, paste the following text into its **main.yml** file:

```shell
---
dependencies: []
```

##### Tasks Folder

For your **node** role, in the **tasks** folder, paste the following text into its **main.yml** file:

```shell
---
- file:
    path=/var/www/html/node
    recurse=yes
    state=directory

- name: Add server.js file  
  copy: src=server.js dest=/var/www/html/node/server.js owner=root group=root

- script: ./install-node.sh
```

___

> ### 9. Githook Role

##### Files Folder

For your **githook** role, in the **files** folder, create a new file named **githook-setup.sh** and paste the following text:

```shell
#!/bin/bash

cd /var

if [ ! -d /var/repos ]; then
  mkdir repos && cd repos
  mkdir html && cd html
  git init --bare
  cd hooks
  touch post-receive
  chmod +x post-receive
  echo "#!/bin/bash" > post-receive
  echo "git --work-tree=/var/www/html/html/ --git-dir=/var/repos/html/ checkout -f" >> post-receive
  cd /var/repos
  mkdir wordpress && cd wordpress
  git init --bare
  cd hooks
  touch post-receive
  chmod +x post-receive
  echo "#!/bin/bash" > post-receive
  echo "git --work-tree=/var/www/html/wordpress/ --git-dir=/var/repos/wordpress/ checkout -f" >> post-receive
  cd /var/repos
  mkdir node && cd node
  git init --bare
  cd hooks
  touch post-receive
  chmod +x post-receive
  echo "#!/bin/bash" > post-receive
  echo "git --work-tree=/var/www/html/node/ --git-dir=/var/repos/node/ checkout -f" >> post-receive
fi
```

This will check if your repo already exists, if not, it will create the new repos and your githooks for post-receive.
##### Meta Folder

For your **githook** role, in the **meta** folder, paste the following text into its **main.yml** file:

```shell
---
dependencies: []
```

##### Tasks Folder

For your **githook** role, in the **tasks** folder, paste the following text into its **main.yml** file:

```shell
---
- script: ./githook-setup.sh
```

___

> ### 10. Playbook File

##### Create Playbook.yml

In the root of your Ansible project, create a **playbook.yml** file and add the following text:

```shell
---
  - hosts: GROUP_NAME
    become: true
    user: root
    roles:
      - nginx
      - html
      - wordpress
      - node
      - githook
```

This will actually execute our roles to run on every group's IPs in our **hosts** file.

___

> ### 11. Ansible Automation

##### Run Ansible Playbook

```shell
ansible-playbook --private-key=~/.ssh/id_ID_NAME -i ./hosts playbook.yml
```

This will run our playbook and automate installing everything we need onto our servers. This may take a while to finish running, but you can watch the progress as it runs.

___

> ### 12. Access Your Sites

Your pipelined sites should now be available at the folloing addresses:

* **http://html.YOUR_IP_ADDRESS.xip.io**
* **http://php.YOUR_IP_ADDRESS.xip.io**
* **http://node.YOUR_IP_ADDRESS.xip.io**
* **http://YOUR_IP_ADDRESS**

___






