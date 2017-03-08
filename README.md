# Repo Deployment
### Deploying files to our servers.

This requires that you've already followed the [setup guide](https://github.com/bbcharlton/DWA/blob/master/setup.md).
___

> ### 1. Local Directory Setup

On your local machine, create a project folder named whatever you want.

##### Git Setup Bash Script

Create a file named **git-setup.sh** and paste the following text:

```shell
#!/bin/bash
mkdir html && cd html
git init
git remote add htmlRepo root@YOUR_IP_ADDRESS:/var/repos/html
cd ..
mkdir php && cd php
git init
git remote add phpRepo root@YOUR_IP_ADDRESS:/var/repos/php
cd ..
mkdir node && cd node
git init
git remote add nodeRepo root@YOUR_IP_ADDRESS:/var/repos/node
cd ..
```

##### Note: Use your Staging server's IP for the remote IP. You can also duplicate this, change the remote name, and use your Production server's IP to allow access to push to it. We are also using root as the user because we used Ansible to automate our servers.

This bash script will create all necessary directories and make them git repos. However we need to make it executable.

```shell
chmod +x git-setup.sh
```

Now that it's executable, let's run it.

```shell
./git-setup.sh
```

Now all directories and repos have successfully been created.

___

> ### 2. Workflow!

##### Use Workflow

Now if you have multiple projects such as HTML, Node, or Wordpress on your local machine, you can then specify which repo you want to send the files to from your local machine and they'll appear in the appropriate directory on either your Staging or Production server.

___
