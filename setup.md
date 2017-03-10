# Ubunutu 16.04/Pipeline Ansible Automation
### Using Digital Ocean to spin up an Ubuntu server that uses two different pipelines, HTML and Node. Each pipeline will include two projects.
___

> ### 1. Generate Your SSH Key

##### Create SSH Key

##### Note: If you already have your own ssh key, you can use that key instead of generating a new one.

```shell
cd ~/.ssh
ssh-keygen -t rsa -b 4096 -C "YOUR@EMAIL_ADDRESS" -f ID_NAME
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
ansible all -m raw -s -a "sudo apt-get -y install python-simplejson" -u root --private-key=~/.ssh/ID_NAME -i ./hosts
```

We are required to use the raw module in order to get Python working on our Ansible automation. This will allow us to use Python for future steps. This may take a while. You will be asked about the authenticity of the host. Enter **y** for yes to add the IP to your known hosts.

##### Ping Our Server

```shell
ansible all -m ping -u root --private-key=~/.ssh/ID_NAME -i ./hosts
```

We should now get two successful outputs from our server pings.

___

> ### 4. Setup Ansible Files

##### Prepare Ansible Files

```shell
mkdir roles
cd roles
mkdir nginx
mkdir node
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

    root /var/www/html/html/proj1;
    index index.html index.htm index.nginx-debian.html;

    server_name html1.{{ ip }}.xip.io;

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

    root /var/www/html/html/proj2;
    index index.html index.htm index.nginx-debian.html;

    server_name html2.{{ ip }}.xip.io;

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

    root /var/www/html/node/proj3;
    index index.html index.htm index.nginx-debian.html;

    server_name node1.{{ ip }}.xip.io;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
    }
}

server {
    listen 80;
    listen [::]:80;

    root /var/www/html/node/proj4;
    index index.html index.htm index.nginx-debian.html;

    server_name node2.{{ ip }}.xip.io;

    location / {
        proxy_pass http://localhost:3001;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
    }
}
```

This will be our Nginx root document that points to specific locations for all six of our our pipelines to work.

##### Vars Folder

For your **nginx** role, in the **vars** folder, paste the following text into its **main.yml** file:

```shell
---
ip: YOUR_IP_ADDRESS
domain: YOUR@DOMAIN_NAME
```

This domain variable should be the same as your **.j2** file's domain name.

___

> ### 6. Node Role

##### Files Folder

For your **node** role, in the **files** folder, create a new file named **install-node.sh** and paste the following text:

```shell
#!/bin/bash

cd ~

if [ ! -d ~/nodesource_setup.sh ]; then
  curl -sL https://deb.nodesource.com/setup_6.x -o nodesource_setup.sh
  bash nodesource_setup.sh
  apt-get install nodejs
  apt-get install build-essential -y
  npm install -g pm2
fi
```

This script downloads Node's source to allow installation of NodeJS. It also installs the build-essential package that is required with Node. Lastly, we install PM2 to allow our apps to continuously run.

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
- script: ./install-node.sh
```

___

> ### 7. Githook Role

##### Files Folder

For your **githook** role, in the **files** folder, create a new file named **githook-setup.sh** and paste the following text:

```shell
#!/bin/bash

cd /var

if [ ! -d /var/repo ]; then
  mkdir repo && cd repo
  git init --bare
  cd hooks
  touch post-receive
  chmod +x post-receive
  echo "#!/bin/bash" > post-receive
  echo "git --work-tree=/var/www/html --git-dir=/var/repo checkout -f" >> post-receive
fi
```

This will check if your repo already exists, if not, it will create the new repo and your githook for post-receive.

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

> ### 8. Playbook File

##### Create Playbook.yml

In the root of your Ansible project, create a **playbook.yml** file and add the following text:

```shell
---
  - hosts: GROUP_NAME
    become: true
    user: root
    roles:
      - nginx
      - node
      - githook
```

This will actually execute our roles to run on every group's IPs in our **hosts** file.

___

> ### 9. Ansible Automation

##### Run Ansible Playbook

```shell
ansible-playbook --private-key=~/.ssh/ID_NAME -i ./hosts playbook.yml
```

This will run our playbook and automate installing everything we need onto our servers. This may take a while to finish running, but you can watch the progress as it runs.

___






