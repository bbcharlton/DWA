# Continuous Integration Deployment
### Using a centralized workflow to continuously integrate and deploy files from Github through Codeship to your server.

This guide requires that you've already followed the [setup guide](https://github.com/bbcharlton/DWA/blob/a345cf2dba767f1f13d905ae9714d51369cf9b2a/setup.md).
___

> ### 1. Clone Github Repos

On your local machine clone down all of your Github project repos to your preferred destination. You will now be able to work locally on the files. For example:

```shell
git clone https://github.com/bbcharlton/StaticRepo1.git
```

___

> ### 2. Edit Project Files

You can use Feature Branch Workflow to create feature branches and edit your projects. This will make sure you don't negatively affect your project if you make a mistake or if your code doesn't function the way you want it to.

___

> ### 3. Continuous Integration

##### Build Test

Now when you're ready to stage your commits and push, the files will be sent to Github. Since we setup Codeship to listen to our repos, it will grab the files and run the builds. You can watch the builds ran from the [projects page](https://app.codeship.com/projects) on Codeship. If a build fails, it will be red, if it passes, it's green. Each build can be reran at anytime. You will also receive an email about the results of the build.

##### Build Success

When a build succeeds, it will send our commit files to our servers. You can test that the files are in the right place by entering the server.

```shell
ssh root@YOUR_STAGING_IP
ssh root@YOUR_PRODUCTION_IP
```

##### Node Projects

For any Node projects added to your server, you will need to go in and npm install its packages, then manually start them with PM2.

```shell
ssh root@YOUR_IP_ADDRESS
cd /var/www/html/PROJECT_REPO
npm install
pm2 start server.js
```

Now our Node apps will be running continuously in the background.

___

> ### 4. Visit Sites

Now that we've used continuous integration to get put our repo projects through our pipelines, we can now visit our project sites at:

```shell
http://PROJECT_NAME.YOUR_IP_ADDRESS.xip.io
```

___
