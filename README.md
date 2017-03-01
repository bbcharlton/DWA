# Wordpress Favicon Setup
### Uploading and setting a favicon for your Wordpress website.
### Note: This guide requires that you've already followed the [setup guide](https://github.com/bbcharlton/DWA/blob/master/setup.md).
___

> ### 1. Create Or Find A Favicon

This requires you to either create your own favicon or use a generated/bought favicon. For ease of this setup, name it **favicon.ico**.

___

> ### 2. Upload Favicon


##### Copy Favicon With SCP

With scp, we'll need to send the file to the remote server's home directory because otherwise we'd get a permission denied error, since the user on your local machine is not the same user as on the remote server.

<pre>

scp -r <span style="color: red">/path/to/favicon.ico</span> <span style="color: red">user</span>@<span style="color: red">your-IP-address</span>:~/

</pre>

Enter your remote user's password and your favicon will now successfully be on the server. Now switch back to your remote server's console.

##### Move Favicon To Directory

<pre>

cd ~
sudo mv favicon.ico /usr/share/nginx/html/wp-content/uploads

</pre>

This moves our favicon to our Wordpress site's uploads file where it can now be read by the server.

##### Add Favicon Code

<pre>

sudo nano /usr/share/nginx/html/wp-content/themes/<span style="color: red">your-theme-folder</span>/header.php

</pre>

We will be entering into the **header.php** file to implement our favicon into the template. Paste the following lines of code into the **\<head>\</head>** element:

<pre>

&lt;link rel="icon" href="http://<span style='color: red'>your-IP-address</span>/wp-content/uploads/favicon.ico" sizes="32x32"&gt;
&lt;link rel="icon" href="http://<span style='color: red'>your-IP-address</span>/wp-content/uploads/favicon.ico" sizes="192x192"&gt;

</pre>

Save and exit out to apply our changes.

___

> ### 3. Refresh Wordpress Page

Now just simply refresh your Wordpress page and you'll see the newly added favicon next to the title of your site!

___

