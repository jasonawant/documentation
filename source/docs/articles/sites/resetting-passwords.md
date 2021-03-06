---
title: Resetting Passwords
description: Learn how to reset your Pantheon Website Management Platform dashboard passwords.
category:
  - managing
keywords: reset password, reset drupal password, reset wordpress password, password, reset
---
If you need to reset your Pantheon Dashboard user password, [visit this page](https://dashboard.pantheon.io/reset-password) and follow the instructions.


## Drupal Site User Login

If you need to reset your Drupal site user login, append `/user/password` to your site's URL and follow the directions to reset your password. For example, to reset the password for the development environment of mysite, you would visit the following example link:
```http
http://dev-mysite.pantheon.io/user/password
```
In the password reset form, enter either the username or email address you used to sign up for the administrative account, and you will receive an email with a link. When you click the link in your email, you will be logged in to your site and brought to your user profile edit page, where you can reset your password. You need to enter your new password at this point. Don’t leave the page without setting a new password, or else you will have to go through this  process again the next time you want to login to your Drupal site.

Please keep in mind that your site password is stored in a database, so whatever you set in the Development environment may be different than Test or live, unless you keep the database content synced between the environments using the Pantheon Dashboard workflow tools or during deployment.

If you still can’t get access to your site using password reset, for example if you don't have access to the corresponding email address for the account, you can still generate a one-time password reset link by using the following [Terminus](https://github.com/pantheon-systems/cli) command for generating one-time login links:

```bash
$ terminus drush --site=<site> --env=<env> user-login
```

<div class="alert alert-info" role="alert">
<h4>Note</h4>Replace <code>&lt;site&gt;</code> with your site name, and <code>&lt;env&gt;</code> with the environment (Dev, Test, or Live). You can see a list of all your sites by running <code>terminus sites list</code></div>

## WordPress User Login
If your site is powered by WordPress you have two options. The first is to request a password reset from the log in form and the second is to update via the [Terminus CLI](https://github.com/pantheon-systems/cli).

1. From the main login form, click **Lost Your Password?**.  
2. Enter your username or password, and click **Get New Password**.

You will receive an email that contains a link you can use one time to reset your password. When you click the link, enter your new password twice. It will also show you the strength of your new password; however, it will not prevent you from using a weak password.

If you have access to the site view Terminus, you can also reset any user's password from the command line.

```nohighlight
$ terminus wp user update \
           --user_pass=NEWPASSWORD \
           --site=YOUR-PANTHEON-SITE-NAME
           --env=dev|test|live
```

As with most terminus commands, you have to give the site's name and the environment you want to make the change in.

As a side note, `terminus wp user update` can be used to change almost any property of a WordPress user's account. `wp_update_user()` gives a complete list of all the firleds that can be changed. You can change any of them by using `--field=value`. In the above command, field is "user_pass" and value is NEWPASSWORD.
