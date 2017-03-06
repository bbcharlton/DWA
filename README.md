# Githook Deployment
### Deploying files and testing them with Githooks.

This guide requires that you have already finished the [setup guide](https://github.com/bbcharlton/DWA/blob/master/setup.md).
___

> ### 1. Local Wordpress Setup

##### Install Wordpress With Docker

<pre>

version: '2'

services:
   db:
     image: mariadb
     volumes:
       - db_data:/var/lib/mysql
     environment:
       MYSQL_ROOT_PASSWORD: <span style="color: red;">password</span>
       MYSQL_DATABASE: wordpressdb
       MYSQL_USER: wpuser
       MYSQL_PASSWORD: <span style="color: red;">password</span>

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8080:80"
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_PASSWORD: <span style="color: red;">password</span>
volumes:
    db_data:

</pre>

Save the above text in a file as **docker-compose.yml**. Move this file into a folder on your desktop. Then open terminal and navigate to this file.

##### Docker Compose

<pre>

docker-compose up

</pre>

This will install Wordpress locally and generate all the files.

##### Install Mocha

<pre>

npm install --save-dev mocha
nano test/test.js

</pre>

If you don't have npm installed, follow [this](http://blog.npmjs.org/post/85484771375/how-to-install-npm) tutorial to get it setup. Otherwise, paste the following text into your **test.js** file:

<pre>

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

</pre>

This is a simple test to check if 1 is equal to 1.

##### NPM Test

<pre>

nano package.json

</pre>

This creates a package.json file that Mocha will read our test from. Paste the following text:

<pre>

{
	"scripts": {
		"test": "mocha"
	}
}

</pre>

This will allow us to do:

<pre>

npm test

</pre>

This will now run our test.

##### Initialize Git

<pre>

git init
git remote add origin root@<span style="color: red;">your-IP-address</span>:/var/repos/wp.git

</pre>

This creates a remote that directs to our **/var/repos/wp.git** bare repo.

##### Pre-Commit Hook

<pre>

npm install pre-commit --save-dev
cd .git/hooks
cp pre-commit.sample pre-commit
cd ../..
nano package.json

</pre>

This installs a pre-commit package that will run certain tests customized in the **package.json** file. We'll now go back and make the **package.json** file. Make sure your file looks like the following:

<pre>

{
  "scripts": {
    "test": "mocha"
  },
  "pre-commit": [ "test" ],
  "devDependencies": {
    "pre-commit": "^1.2.2"
  }
}

</pre>

So now when pre-commit is fired, it will call the test script.

##### Post-Merge Hook

<pre>

sudo nano .git/hooks/post-merge

</pre>

This will now create our post-merge hook. Paste the following code into the file:

<pre>

#!/bin/bash
git branch > myGitBranch.txt

if grep -q "* release" myGitBranch.txt; then
	echo 'Merging onto release branch!';
	git checkout master
	git merge release
	git push deploy master
fi

</pre>

This will check the current branch you're on, and if it's the release branch being merged, it will push up the changes to the master branch on the remote server.

##### Make Executable

<pre>

chmod +x .git/hooks/post-merge

</pre>

This will allow post-merge hook to be executable when it's fired.

##### Create Gitignore 

<pre>

cd ~/Desktop/<span style="color: red;">Project-Folder</span>

</pre>

Go back into the root of your local Wordpress project folder, and add a .gitignore file. Paste the following text into that file:

<pre>

/node-modules
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

</pre>

This will make sure that no sensitive information is sent to our remote repo.

___

> ### 2. Pipeline Complete!

##### Use Workflow

Now anytime you make add a file and commit, our test will be ran. If the test fails, the commit doesn't go through. If the test passes, the commit is created. When you're ready to merge your release branch, our post-merge hook will look if we're merging the release branch then push to the remote's master branch.

___
