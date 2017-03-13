# Repo Deployment
### Deploying files to our servers.

This guide requires that you've followed the [setup guide](https://github.com/bbcharlton/DWA/blob/2e434cea72d1e3c046fa2bb5d2f69586f0b5576d/setup.md).
___

> ### 1. Local Directory Setup

On your local machine, download the project repo from my [Github repo](https://github.com/bbcharlton/AnsibleProjectRepos). These files will hold the projects we will deploy. Unzip the file, move the folder to your preferred location, and go into the folder.

##### Git Remote Setup

```shell
git init
git remote add REMOTE root@YOUR_IP_ADDRESS:/var/repo
```

This initializes our folder as a working directory then creates the remote that we'll use to deploy our files.

##### Note: Use your Staging server's IP for the remote IP. You can also duplicate this, change the remote name, and use your Production server's IP to allow access to push to it. We are also using root as the user because we used Ansible to automate our servers.

##### Deploy

```shell
git add .
git commit -m "First commit"
git push origin master
```

This will send our files to our remote server, which will then finalize our folder structure on the server.

##### Verify Deployment

```shell
ssh root@YOUR_IP_ADDRESS
cd /var/www/html
```

We'll ssh into our server then should see our project folders: html and node.

___

> ### 2. PM2 Start Our Projects

##### Node Projects

```shell
cd node
cd proj3
npm install
npm install express-validator
npm install pug
pm2 start server.js
```

This will prepare the packages for our first Node application, then server it using PM2. We must npm install the packages first since we use .gitignore to prevent the transfer of the node_modules folder.

```shell
cd ..
cd proj4
pm2 start server.js
```

Since this Node app doesn't have any node modules, it just needs a simple pm2 start.

___

> ### 3. Visit Subdomains!

##### HTML Subdomains

We can now visit the following sites:

* **html1.YOUR\_IP\_ADDRESS.xip.io**
* **html2.YOUR\_IP\_ADDRESS.xip.io**

And our two HTML projects should be loaded up!

##### Node Subdomains

We can now visit the following sites:

* **node1.YOUR\_IP\_ADDRESS.xip.io**
* **node2.YOUR\_IP\_ADDRESS.xip.io**

And our two Node projects should be loaded up!

___

> ### 4. Use Workflow

##### Edit Local Files

Enter into the repo's folder that you pulled from my Github. Any time you edit any of the projects, you just have to be in the root of the repo file to use basic git workflow commands such as:

```shell
git add .
git commit -m "YOUR MESSAGE"
git push REMOTE master
```
___
