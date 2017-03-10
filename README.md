# Wordpress File Deployment
### Deploying files to your Wordpress site.

This tutorial requires that you've followed the [setup guide](https://github.com/bbcharlton/DWA/blob/75c8120804d60cc26bc18c95161264b04f20b8af/setup.md).
___

> ### 1. Upload File


##### Copy File With SCP

We will use the **scp** command to deploy files from our local machine to our remote server.

```shell
scp -r /PATH/TO/FILE root@YOUR_IP_ADDRESS:/usr/share/nginx/html
```

##### Note: You can change the remote server's directory you're sending the file to based on wherever you need that file sent to. But for this guide, we're sending files to our Wordpress' root directory.

Run the command on your local machine. Your file should now securely transfer to your remote server.

___

> ### 2. Check Remote Directory

Now switch back to your remote server's console and ssh into it.

```shell
ssh root@YOUR_IP_ADDRESS
```

Head to your Wordpress' root directory and the file should now be in your directory.

___

