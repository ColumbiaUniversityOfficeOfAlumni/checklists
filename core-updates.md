Note:
----
these instructions _do not_ apply to a multisite installation. if you want to update core there, I have no advice, but will buy you a drink. -EY

- [ ] Make a backup of your Drupal instance. (I believe all the Alumni and Development sites have [Backup and Migrate](https://www.drupal.org/project/backup_migrate) installed, which is a nice GUI for downloading a zip or tarbell of the database and/or codebase and/or files. It's easy to install if it isn't already there. Otherwise, you can use [mysqldump](http://www.thegeekstuff.com/2008/09/backup-and-restore-mysql-database-using-mysqldump/) or whatever tool you prefer.) Make sure to snapshot the modules page, so you know what's enabled -- updating modules or core sometimes disables modules, seemingly at random, and it can be incredibly frustrating to troubleshoot "why isn't this working?" when it's just that something is mysteriously turned off.
- [ ] Download the [latest release of your current Drupal version](https://www.drupal.org/project/drupal).
- [ ] Extract the [tar ball or zip] Drupal package.
- [ ] If you're working with a site that uses svn to manage version control, check out a working copy of the repo from maple.
- [ ] Diff files such as .htaccess, robots.txt, sites/default/settings.php. in your existing codebase and make sure any customization, including database name, user, and password in settings.php, doesn't get erased when you ....
- [ ] Replace all the folders and files _except /sites_ inside your original Drupal instance with the folders and files in the newer Drupal codebase.
- [ ] Commit the changes.
- [ ] Set your site on maintenance mode at /admin/settings/site-maintenance (D6) or /admin/config/development/maintenance (D7).
- [ ] Update the remote working copy with svn update.
- [ ] Login to your site as user=1, at example.columbia.edu?q=user.
- [ ] Run update.php by navigating to example.columbia.edu/update.php
- [ ] Follow the process to update your Drupal instance
- [ ] Clear site cache, either with drush cc all or through the Performance page in the admin GUI, at /admin/settings/performance (D6) or /admin/config/development/performance (D7)
- [ ] Disable maintenance mode.