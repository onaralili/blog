---
title: "CKAN - open-source platform for your datasets. Installing and Troubleshooting "
date: 2018-07-22
tags: ["ckan", "dataset", "open-source"]
draft: false
repo: "https://www.onaralili.com/posts/ckan"
---

## CMS for open data?
[CKAN](https://ckan.org/) is a free & open-source platform to publish and host datasets in your own server. It has great search integration using [Solr](https://lucene.apache.org/solr/), API, data visualization and many more features that makes open data discoverable. The full list of features can be found [here] (https://ckan.org/features/). In order to install CKAN on your server, you have two options either from source (via GitHub repository) or from the package if you are using Ubuntu. Although official documentation demonstrates on Ubuntu, it works pretty much every Linux distribution, even in Windows. I am going to write about my experience deploying it on Ubuntu 16.04 64-bit server and the problems I faced and the way I solved them.

### Issue #1 Nginx

I followed the official [documentation](http://docs.ckan.org/en/latest/maintaining/installing/install-from-package.html) on the website, however, I encountered some issues. 
Installation went smoothly until I reached the point where I have to install nginx
``` sudo apt-get install -y nginx ```
however, the following error returned during the installation 
```
You might want to run 'apt-get -f install' to correct these: The following packages have unmet dependencies:
...
```
Even though I run *apt-get -f install*, it didn't solve the problem. Following query solve it and installed all missing packages
``` sudo apt --fix-broken install ```
 
### Issue #2 CKAN Configuration and database connection string
Unfortunately, the documentation lacks to mention that you have to modify CKAN config file to be able to connect to the database. Otherwise during the phase :
``` sudo ckan db init ```
You will not be able to initialize the database. In order to config, go to */etc/ckan/default/production.ini* directory. Under the Database Settings, modify the value of *sqlalchemy.url* accordingly, make sure that change parameter *pass* to your database user's password.  After this modification, you will be able to run the query without any issue.

### A few important notes
After the successful installation, you have to create a sysadmin account in order to get admin privileges on the platform. Follow this [doc](http://docs.ckan.org/en/latest/maintaining/getting-started.html#create-admin-user) to add an admin user.  The admin dashboard is under */ckan-admin* directory.

