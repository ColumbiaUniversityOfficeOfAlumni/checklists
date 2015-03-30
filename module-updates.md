This process is specific to the existing CAA installation. If you are not a developer in the Columbia University Office of Alumni and Development's Web Initiatives team, please don't use this as a guide. If you are, this document is accurate to the best of my knowledge and belief as of 30 March 2015, but the ways of the CAA website are strange and frightening, and I frankly am not an expert in SVN. And sometimes Drupal just... does stuff. I don't know either.

Nota bene:
=========
- Do _not_ upgrade Views to 6.x-3.x; we're stuck with the 6.x-2.x branch.
- [Important security note, available only to Lionmail users.](https://docs.google.com/a/columbia.edu/document/d/1omS5MJHfdBVy-85pY1ubE-6---QbQ2dkoO0brrg-z_A/edit?usp=sharing)
- Upgrade modules one at a time. It will be very tempting to do several at once. IGNORE THIS IMPULSE.
- Before beginning the upgrade process, back up your database and codebase (the latter _outside_ of SVN, because I hate, fear, and distrust SVN.)
- Take a snapshot of what modules are enabled -- updating modules or core sometimes disables modules, seemingly at random, and it can be incredibly frustrating to troubleshoot "why isn't this working?" when it's just that something is mysteriously turned off.
- On eng, sites/alumni.columbia.edu is an alias/symbolic link for sites/dev.alumni.columbia.edu; sites/magazine.columbia.edu is an alias/symbolic link for sites/dev.magazine.columbia.edu -- do your work in the dev folders. This is an artifact of the development process lo these many years again and there is an eldritch god that will heap retribution upon your head if you interfere with this structure.

Okay. Let's begin.
-----------------

Check out a working copy of the SVN repo from maple to your local. Not on eng. There are two folders in trunk: modules and themes.

Choose the module you want to update from the list of available updates at /admin/reports/updates.

If you are using drush dl <modulename>, specify that you want the 6.x branch; I've had drush go ahead and grab the 7.x branch _just to make me crazy_.
	
Commit changes to your local working copy. Write a clear, concise commit message. (Yes, we all know [What the Commit](http://whatthecommit.com/) and [Commits from Last Night](http://www.commitlogsfromlastnight.com/) (warning for profanity). Your commit message will be funny for five-point-two minutes and then it will live forever.)

Log onto eng. Open D:\mirror\drupal sites\acquia-drupal-1.2.21\sites\dev.alumni.columbia.edu\modules.

Right-click and open the TortoiseSVN client. Find the module you want to update. Check it out to the eng local.

Make sure the site knows about the new module version -- I've never quite figured out when you just need to svn co the module and when you need to svn up as well. You may need to clear the site cache as well.

And now it's time for the really fun part: updating the database.

- [ ] Set the site to maintenance mode at /admin/settings/site-maintenance
- [ ] Login to the site as user=1, at alumni.columbia.edu/user.
- [ ] Run update.php by navigating to alumni.columbia.edu/update.php
- [ ] Follow the process to update the database
- [ ] Clear site cache, either with drush cc all or through the Performance page in the admin GUI, at /admin/settings/performance
- [ ] Disable maintenance mode