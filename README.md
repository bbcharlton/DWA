# Githook Deployment
### Deploying files and testing them with Githooks.

This requires that you've followed the [setup.md guide](https://github.com/bbcharlton/DWA/blob/4d892cbebb893fd6d1958b83c7338656356c8215/setup.md). Before we start, here's a general diagram of how our workflow will go:

![Alt text](http://imgur.com/a/t9sSH)

___

> ### 1. Local Wordpress Setup

##### Install Wordpress With Docker

```shell
version: "2"
services:
  my-wpdb:
    image: mariadb
    ports:
      - "8081:3306"
    environment:
      MYSQL_ROOT_PASSWORD: ROOT_PASSWORD
  my-wp:
    image: wordpress
    volumes:
      - ./:/var/www/html
    ports:
      - "8080:80"
    links:
      - my-wpdb:mysql
    environment:
      WORDPRESS_DB_PASSWORD: DATABASE_PASSWORD
```

Save the above text in a file as **docker-compose.yml**. Move this file into a folder on your desktop. Then open terminal and navigate to this file.

##### Docker Compose

```shell
docker-compose up -d
```

This will install Wordpress locally and generate all the files.

##### Install Mocha

```shell
npm install --save-dev mocha
mkdir test && cd test
nano test.js
```

If you don't have npm installed, follow [this](http://blog.npmjs.org/post/85484771375/how-to-install-npm) tutorial to get it setup. Otherwise, paste the following text into your **test.js** file:

```shell
var assert = require('assert');
describe('Array', function() {
  describe('Unit Test', function() {
    it('Should check if 1 == 1', function(done) {
      if (1 == 1) {
      	done();
      } else {
      	console.log('Test failed! Commit not sent!');
      }
    });
  });
});
```

This is a simple test to check if 1 is equal to 1.

##### Initialize Git

```shell
cd ..
git init
git remote add origin root@YOUR_IP_ADDRESS:/var/repo
```

This creates a remote that directs to our **/var/repo** bare repo.

##### Pre-Commit Hook

```shell
npm install pre-commit --save-dev
nano package.json
```

This installs a pre-commit package that will run certain tests customized in the **package.json** file. We'll now create the **package.json** file. Paste the following text:

```shell
{
  "scripts": {
    "test": "mocha"
  },
  "pre-commit": [ "test" ],
  "devDependencies": {
    "pre-commit": "^1.2.2"
  }
}
```

So now when pre-commit is fired, it will call the test script.

##### Post-Merge Hook

```shell
nano .git/hooks/post-merge
```

This will now create our post-merge hook. Paste the following code into the file:

```shell
#!/bin/bash

BRANCH=$(git rev-parse --abbrev-ref HEAD)
if [[ "$BRANCH" == "release" ]]; then
	echo 'Merging onto' $BRANCH 'branch!';
	git push origin master
fi
```

This will check the current branch you're on, and if it's the release branch being merged, it will push up the changes to the master branch on the remote server.

##### Make Executable

```shell
chmod +x .git/hooks/post-merge
```

This will allow post-merge hook to be executable when it's fired.

##### Create Gitignore 

Create a **.gitignore** file to prevent sending unecessary files and information. Paste the following text into that file:

```shell
/node-modules
/test
docker-compose.yml
package.json
*.log
wp-config.php
wp-content/advanced-cache.php
wp-content/backup-db/
wp-content/backups/
wp-content/blogs.dir/
wp-content/cache/
wp-content/upgrade/
wp-content/uploads/
wp-content/wp-cache-config.php
wp-content/plugins/hello.php
/.htaccess
/license.txt
/readme.html
/sitemap.xml
/sitemap.xml.gz
```

This will make sure that no sensitive information is sent to our remote repo.

___

> ### 2. Pipeline Complete!

##### Use Workflow

Now anytime you add a file and commit, our test will be ran. If the test fails, the commit doesn't go through. If the test passes, the commit is created. When you're ready to merge your release branch, our post-merge hook will look if we're merging the release branch then push to the remote's master branch.

___
