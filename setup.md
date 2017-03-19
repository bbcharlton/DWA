# Ubunutu 16.04/Pipeline Ansible Automation
### Using Digital Ocean to spin up an Ubuntu server that uses multiple pipelines and a centralized workflow.
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
cat ID_NAME.pub
```

This outputs our public ssh key. Copy this key. We will need it for our Digital Ocean steps.

___

> ### 2. Setup Codeship App(s)

Create an account on [Codeship](https://codeship.com/) by connecting your Github account. Once logged in, create a new project and link your preferred Github repo(s). This will be the central point of our centralized workflow. Any commits pushed to this repo will then be deployed to our remote server. Select the Basic package for your account when prompted. 

We'll now need the project's SSH key. To obtain it, go to your Codeship project's settings and click **General**. Scroll down and copy the SSH key.

___

> ### 3. Creating Your Droplet With SSH

Sign in to your Digital Ocean account and go to your settings page. Click on **Security** and click the **Add SSH Key** button. Paste in your copied Codeship public key as well as your personal ssh key and give them unique names. Save and go to the Droplets page. Customize how you want your Droplet to be. Be sure to use **Ubuntu 16.04 x64** for this specific setup. For the SSH option, click on the SSH key name you just added to your profile, and create.

If you have multiple Codeship projects, you will need to add each ssh key to your droplet.

___

> ### 4. Setup Deployment

After creating your Codeship project, go to your project's settings and click **Deployment**. Set the branch to be used as **master**. Save and then you'll be directed to choose a deployment for your pipeline. Choose custom script and paste the following command:

```shell
rsync -avz -e "ssh" /home/rof/src/github.com/GITHUB_USERNAME/YOUR_REPO root@YOUR_STAGING_IP:/PATH/TO/REPO/WORKING/DIRECTORY
rsync -avz -e "ssh" /home/rof/src/github.com/GITHUB_USERNAME/YOUR_REPO root@YOUR_PRODUCTION_IP:/PATH/TO/REPO/WORKING/DIRECTORY
```

This will be the command that will send the files we push to our Github branch to our remote servers. This can be used for multiple Codeship projects.

___

> ### 5. Install Ansible With Brew

##### Install Brew

You can use the following article on how to install [Brew](https://brew.sh/).

##### Install Ansible

```shell
brew install ansible
```

This installs Ansible globally on our local machine, which allows us to use it for any local project.

___

> ### 6. Download Ansible File Structure

##### Prepare Ansible Files

Either clone or download the Ansible file structure from my [Github repo](https://github.com/bbcharlton/AnsibleTemplate) onto your local machine in your preferred location.

##### Ping Our Servers

```shell
ansible staging -m ping -u root --private-key=~/.ssh/ID_NAME -i ./hosts
ansible production -m ping -u root --private-key=~/.ssh/ID_NAME -i ./hosts
```

You will be asked about the authenticity of the host. Enter **y** for yes to add the IP to your known hosts for both commands. We should get errors back from both pings because we do not have the right version of Python installed.


##### Install Python

```shell
ansible all -m raw -s -a "sudo apt-get -y install python-simplejson" -u root --private-key=~/.ssh/ID_NAME -i ./hosts
```

We are required to use the raw module in order to get Python working on our Ansible automation. This will allow us to use Python for future steps. This may take a while.

___

> ### 7. Ansible Automation Instruction

The Ansible file structure has 6 roles:

* **server**
* **nginx**
* **node**
* **wordpress**
* **createAdmin**
* **createDBAdmin**

> ##### 1. Server Role

This role is used to register a new server document. It has a **vars** folder with a **main.yml** file. For the variables, set the **tech** variable to one of the following:

* **html**
* **php**
* **node**

This will notify Ansible on which server document to use for the project creation. For the **project** variable, set its value to the exact name of your project repo you'll be adding. Lastly, if you're adding a Node app, set the **port** variable to the port number you want your reverse proxy to use. Here's an example of how your **main.yml** file should look like:

```shell
---
tech: html
project: StaticRepo1
port:
```

But in order for our Ansible automation to use the server document correctly, we must also rename it our project repo's name. For example, we have the **html** document in our **templates** folder as:

```shell
html-NAME
```

Simply replace name with your project repo's name. For example:

```shell
html-StaticRepo1
```

Now our server document references should be all ready to go. This entire **server** section must be completed per project you're adding to your server.

> ##### 2. Nginx Role

This role is used to download and install Nginx on the server. This only needs to be ran once to initialize the server's Nginx files.

> ##### 3. Node Role

This role is used to download and install Node on the server. Along with Node, it will also install PM2 to allow our Node apps to run continuously in the background. This only needs to be ran once to initialize the server's Node files. Whenever you have your Node app in your working directory, don't forget to install its packages with:

```shell
npm install
```

The packages should download. Then you can start your Node app with PM2.

```shell
pm2 start server.js
```

> ##### 4. Wordpress Role

This role is used to download and install Wordpress on the server. Along with Wordpress, it will also install MariaDB to store its data. It has a **vars** folder with a **main.yml** file. For the **project** variable, set its value to the exact name of your project repo you'll be adding. You can change the **wp_version** to any Wordpress version but 4.7.3 is recommended. Lastly, change the database variables to what you want your information to be. Here's an example of how your **main.yml** file should look like:

```shell
---
project: WordpressRepo1
wp_version: 4.7.3
wp_db_name: DATABASE_NAME
wp_db_user: DATABASE_USERNAME
wp_db_password: DATABASE_PASSWORD
auto_up_disable: false
core_update_level: true
```

> ##### 5. CreateAdmin Role

This role is used for creating admin users for the **sudo** group. These users are considered admins because they have root access through the sudo command. It has a **vars** folder with a **main.yml** file. For the variables, set the user's name and password and give it a ssh key to use to login.

> ##### 6. CreateDBAdmin Role

This role is used for managing databases. You must specify exactly which database this user can have access to. It has a **vars** folder with a **main.yml** file. For the variables, set the user's name and password and give it a ssh key to use to login. Then specify the name of the database they will have access to.

___

> ### 8. Playbook File

##### Configure Playbook.yml

```shell
---
  - hosts: staging:production
    become: true
    user: root
    roles:
      - nginx
      - server
      - node
      - wordpress
      - createAdmin
      - createDBAdmin
```

The playbook is what decides which roles you'll be applying to your servers. It's completeley modular, so if you want to add multiple server documents for your servers, simply just add the **server** role and refer to the section on its details above.

___

> ### 9. Ansible Automation

##### Run Ansible Playbook

```shell
ansible-playbook --private-key=~/.ssh/ID_NAME -i ./hosts playbook.yml
```

This will run our playbook and automate installing everything we need onto our servers. This may take a while to finish running based on your roles used, but you can watch the progress as it runs.

___

> ### 10. Activity Monitor

##### Optional Step!

If you want to monitor all the activity and processes on your server, HTOP is a pretty simple and neat service to use. 

```shell
ssh root@YOUR_IP_ADDRESS
apt-get install htop
```

This will install HTOP onto your server. At any time, you can run the command:

```shell
htop
```

This will bring up your activity monitor. To exit out press **Ctl + C**. For more information about HTOP, [visit the official website](http://hisham.hm/htop/index.php).

___