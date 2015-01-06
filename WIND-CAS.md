# How to move a Drupal site from WIND to CAS, for Columbia University sites

This document assumes that you have access to your site's database and codebase. It will work for both Drupal 6 and 7, with small variations in admin URLs. I've tried to be as detailed as possible.

## Before beginning, read through [CAS Authentication at Columbia University](http://cuit.columbia.edu/cas-authentication).

- [ ] File a [Service Registration Request](http://cuit.columbia.edu/cas-service-registration-request) with CUIT's Identity Management, either via email or ServiceNow. They're generally very good at responding within 24 hours, but I'd give them 48 before panicking. Once you've got that all sorted out:
- [ ] Download the [CAS module](https://www.drupal.org/project/cas) to your local sites/all/modules (or sites/all/modules/contrib, if that's how you roll). Unzip.
- [ ] Download the most current version of the [phpCAS library](https://wiki.jasig.org/display/CASC/phpCAS) to your local sites/all/libraries. Unzip.
- [ ] Make sure your .htaccess file has the following code in its Rewrite Module:
```
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```
- [ ] Deploy the module, the library, and the .htaccess file to your server.
- [ ] Enable the CAS module on your site (don't bother with the CAS server submodule). Drupal 6: admin/build/modules. Drupal 7: admin/modules.
- [ ] Enable "administer cas" permission for whatever role administers the site. Drupal 6: admin/user/permissions. Drupal 7: /admin/people/permissions.
- [ ] Go to the CAS settings page. Drupal 6: admin/user/cas. Drupal 7: admin/config/people/cas.
- [ ] The library directory is `sites/all/libraries/[CAS folder]`.
- [ ] The CAS server version is 2.0; I know CUIT/IDM has SAML capabilities, but I've never used it and can't say if that changes anything. The hostname is `cas.columbia.edu`. The port is 443. The URI is /cas. Leave the Certificate Authority blank.
- [ ] I generally tweak the login form settings; depending on the site, you may want to make CAS login the default on login forms. Change the CAS login invitation to `Log in with your UNI` -- this will save confusion about what CAS is versus UNI. Change the Drupal login invitation to something that makes it clear it is _not_ UNI authentication. The redirection notification message should make it clear that you're redirecting people to a UNI authentication page. You may want to remove references to CAS in the successful login message.
- [ ] User accounts settings: check the box to automatically create Drupal accounts. The email address should be username@`columbia.edu`. Choose whatever roles are relevant to your site. Both the "Users cannot change email address" and "Users cannot change password" checkboxes should be checked.
- [ ] The settings for the redirection, login/logout destinations/and miscellaneous & experimental settings can be left as default.
- [ ] Save configuration.
- [ ] Put the site in maintenence mode, since you're about to be messing with the database, if only briefly.
- [ ] Connect to your site's database with your preferred MySQL client and execute the following query:
```
INSERT INTO cas_user (uid, cas_name)
SELECT u.uid AS uid, u.name as cas_name
FROM users u
WHERE uid <> 0 AND NOT EXISTS (SELECT cas_name FROM cas_user c WHERE c.cas_name = u.name);
```
- [ ] If you look at the Users page, you should see something similar to the following:
1. Drupal 6
![Drupal 6](https://www.evernote.com/shard/s10/sh/68fe61d7-6acb-4108-9c2f-c8d4cf8384bd/ecbbede72b11003f483a2af6909443da/deep/0/Drupal-6-CAS-users.png)
1. Drupal 7
![Drupal 7](https://www.evernote.com/shard/s10/sh/aaf37765-cf4b-475f-ab45-015ce68d5795/98013744cf8535c37129489d22ddce14/deep/0/Drupal-7-CAS-users.png)
- [ ] Change all your login links to https://yoursite.columbia.edu/cas. Disable the WIND module, which you no longer need. Add a redirect from /user/wind to /cas so users don't get a 404. (Make sure that the redirection is going to an https page or you will get a scary error message when you try to log in.)
- [ ] Take the site out of maintenance mode.
- [ ] Log out and log back in with your shiny new CAS login and give yourself a high-five.