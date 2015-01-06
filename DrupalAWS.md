Assumptions:
- you have a .sql backup of an existing site's database
- you have a backup of any custom themes and modules used in an existing site
- you are comfortable with git and the command line

Spin up a new site
------------------
- [ ] [Launch a new EC2 instance](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-instance_linux.html)
- [ ] [Connect to the instance using PuTTY or SSH](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-connect-to-instance-linux.html)
- [ ] On the command line, run the following installs (this is for Red Hat and will need to be updated for Ubuntu):
```
sudo yum -y update
sudo yum -y install httpd mysql mysql-server php php-cli php-gd php-intl php-mbstring php-mysql php-pdo php-pear php-xml php-xmlrpc
```
- [ ] Make sure services automatically start on a reboot:
```
sudo chkconfig httpd on
sudo chkconfig mysqld on
sudo service httpd start
sudo service mysqld start 
```
- [ ] Edit your httpd.conf:
```
sudo cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.old
sudo vi /etc/httpd/conf/httpd.conf 
```
Find ```<Directory "/var/www/html">``` and under that change ```AllowOverride None``` to ```AllowOverride All```

Press Ctrl + C and type :wq to save and quit.
- [ ] Restart apache for changes to take effect:
```
sudo service httpd restart 
```
- [ ] Set your mysql root password
```
sudo mysqladmin -u root password 'new-password' 
```
- [ ] Set mysql security:
```
sudo mysql -u root -p
mysql> DROP DATABASE test; 
mysql> DELETE FROM mysql.user WHERE user = ''; 
mysql> FLUSH PRIVILEGES; 
```
- [ ] Create the database that drupal will use:
```
mysql> CREATE DATABASE databasename;
```
- [ ] Press Ctrl + C to exit mysql.
- [ ] Install Drupal via drush. Use the most recent version number of Drupal 6 or 7, available at [drupal.org](http://drupal.org/project/drupal); as of this writing, that would be 7.34 and 6.34.
```
sudo pear channel-discover pear.drush.org
sudo pear install drush/drush
cd /var/www/html/
sudo drush dl
sudo mv drupal-7.x/* ./
sudo mv drupal-7.x/.htaccess ./
sudo rm -r drupal-7.x 
```
- [ ] Create the files directory and the settings.php file.
```
sudo mkdir sites/default/files
sudo chmod 775 sites/default/files/
sudo cp sites/default/default.settings.php sites/default/settings.php
sudo chmod 777 sites/default/settings.php
```
- [ ] Update the sites/default/settings.php file:
Drupal 6:
```
$db_url = 'mysql://username:password@endpoint/databasename';
```
Drupal 7:
````
$databases = array (
  'default' => 
  array (
    'default' => 
    array (
      'database' => 'databasename',
      'username' => 'username',
      'password' => 'password',
      'host' => 'endpoint',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
````
Save.
- [ ] Change permissions on settings.php
```
sudo chmod 444 sites/default/settings.php 
```
- [ ] Find the public DNS listed in the AWS for the instance (e.g., ```ec2-00-00-000-00.compute-1.amazonaws.com```). Open this address in a browser. Follow the instructions in the site.

Congratulations, you now have a working empty Drupal site.
-----------------------------------------------------------------------

Set up deploy with git
----------------------------
- [ ] copy your public key to your ec2 instance:
```
cat ~/.ssh/id_rsa.pub | ssh -i ~/.ssh/your_pemfile.pem username@your_ip_addr "cat>> .ssh/authorized_keys"
```
- [ ] Use ssh or PuTTY to connect to the instance and create a bare git directory:
```
cd ~
mkdir ProjectDir.git && cd ProjectDir.git
git init --bare
```
- [ ] Create a post-receive hook in the bare repo:
```
nano hooks/post-receive
```
Put the following into the file (using the correct path, obviously) and save:
```
#!/bin/sh
GIT_WORK_TREE=/var/www/html/example
export GIT_WORK_TREE
git checkout -f
```
- [ ] Make the hook executable:
```
chmod +x hooks/post-receive
```
- [ ] On your local computer, add the EC2 repo as a [remote](http://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes) you can push to:
```
git remote add ec2 ssh://username@your_ip_addr/home/ProjectDir.git
```
- [ ] Push an up-to-date version of the codebase from your local repo:
```
git push ec2 +master:refs/heads/master
```
Note: you only have to use ```+master:refs/heads/master``` the first time; after that it's the normal ```git push ec2 master```.

- [ ] To push a project to Github and EC2 at the same time (**recommended**), open .git/config in your local repo (use a plain text editor: nano, TextEdit, TextMate, etc.), and add the following (edited for accuracy):
```
[remote "all"]
        url = https://github.com/ColumbiaUniversityOfficeOfAlumni/ProjectDir.git
        url = ssh://username@your_ip_addr/home/ubuntu/projects/ProjectDir.git
```
You can now ```git push all master``` and it will automagically happen.

Import the database backup to the new site
-------------------------------------
- [ ] Use ssh or PuTTY to connect to the instance and import the .sql dump:
```
mysql -u username -p -h endpoint databasename < backupfile.mysql
```
- [ ] Check the site is up. Check the status report.

Starting points for troubleshooting:
==============
- .htaccess files
- Clean URLs
- PHP settings
When importing the database, if you get ```ERROR 2003 (HY000): Can't connect to MySQL server on '' (110)```, try adding :3306 to the endpoint.
