# Centralized Workflow Deployment
### Deploying files to our servers.

This guide requires that you've already followed the [setup guide](https://github.com/bbcharlton/DWA/blob/0c325e7a7121560e803177d471e3c9af5bdb55b9/setup.md).
___

> ### 1. Clone Github Repo

##### Pull Codeship Repo

```shell
mkdir DIRECTORY_NAME
cd DIRECTORY_NAME
git clone https://github.com/bbcharlton/CodeshipRepo.git
git remote add origin https://github.com/bbcharlton/CodeshipRepo.git
```

Create a new directory on your local machine and clone the CodeshipRepo repo to have access to the files we'll be deploying.

___

> ### 2. Make Changes

##### Edit Repo Files

You can now edit the repo's files. Once you're ready to send the new code to your remote server, just push to the Github repo.

```shell
git add .
git commit -m "COMMIT_MESSAGE"
git push origin master
```

This will then send the files to Github. When Github receives these changes, Codeship will spin up a build and test our code. If the build is successful, it will push the new files to our remote server.

##### PM2 Start Node App

```shell
ssh root@YOUR_IP_ADDRESS
cd /var/www/html/CodeshipRepo/node
pm2 start server.js
```

We require PM2 to keep our Node app running in the background, so we must start it up. Any changes it gets through Codeship will be applied live.

___

> ### 3. Centralized Workflow

##### Use Workflow

We use Github as the central point of our workflow. We pull code from Github, edit the code, and push it back up. Codeship takes the new code from our Github repo and deploys it to our remote server. You can now access your sites at the following domains:

* **http://html.YOUR\_IP\_ADDRESS.xip.io**
* **http://node.YOUR\_IP\_ADDRESS.xip.io**
* **http://php.YOUR\_IP\_ADDRESS.xip.io**
* **http://YOUR\_IP\_ADDRESS**

If your sites are not appearing, you may need to restart Nginx.

```shell
ssh root@YOUR_IP_ADDRESS
systemctl restart nginx
```

This will restart Nginx and your sites should now be serving if they weren't previously.

___
