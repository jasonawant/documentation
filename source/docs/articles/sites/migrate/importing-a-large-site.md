---
title: Migrating to Pantheon: Manual Site Import
description: Learn how to import a Drupal or WordPress site into Pantheon outside of the Dashboard.
keywords: import, importing site, pantheon, new site, large site, distro, upstream
---

When to use Manual Site Import:

* **Large Site Archives**: If a site archive is greater than the automated import limits (100MB for direct file upload or 500MB for URL upload).
* **Custom Upstream**: Site should receive updates based on an upstream other than vanilla Drupal or WordPress (e.g Panopoly, or your agency's customized WordPress).

If your existing site is based on vanilla Drupal or WordPress and has smaller site archives, follow the instructions below or use [automated import](/docs/articles/sites/migrate/#import-archives).

## Requirements

* [Git](/docs/articles/local/starting-with-git/)
* [Rsync or SFTP Client](https://pantheon.io/docs/articles/local/rsync-and-sftp/)
* [MySQL Client](https://pantheon.io/docs/articles/local/accessing-mysql-databases/)

## Create a New Pantheon Site and Start from Scratch 

From your Pantheon Dashboard:

* Choose **Create a new site**.
* Name your site.
* Select **Start from scratch**, and choose your starting codebase.

Starting from scratch allows your site to connect to that upstream so you can later [apply upstream updates](/docs/articles/sites/code/applying-upstream-updates/) from your Dashboard with one click.

## Import the Codebase

**Codebase** - all executable code, including core, custom and contrib modules or plugins, themes, and libraries.

As long as you've chosen the same codebase (Drupal 7, Commerce Kickstart, etc.) as the starting point of your Pantheon site, you can use Git to import your existing code and commit history. If you don’t have a Git version controlled codebase, the following will still work.

1. Go to your code directory within your terminal.
2. If your existing site code is not version controlled with Git, run: ```git init```
3. From the Site Dashboard, go to the **Dev** environment.
4. Switch the site's connection mode from SFTP to Git.
5. Get the upstream's Git connection string:
 - From the Site Dashboard: Click **Settings** >> **About Site**.
 -  Place your mouse over the upstream value, left click and select **Copy link** to get the site's Pantheon upstream location.  
 ![](/source/docs/assets/images/pantheon-dashboard-settings-about-site-upstream.png)  
 - Replace "http" with "git" and then add ".git" to the end of the URL you just copied. For example, if your site is based on Drupal 7 upstream, the URL will go from this: `http://github.com/pantheon-systems/drops-7 to 'git://github.com/pantheon-systems/drops-7.git'
6. Use Git to pull in the upstream's code (which may have Pantheon-specific optimizations) to your existing site's codebase, using the string you created in step 5:

For example, if your upstream is Drupal 7 running:
```bash
git pull --no-rebase -Xtheirs --squash git://github.com/pantheon-systems/drops-7.git master
```  

Will output:  
```bash
Squash commit -- not updating HEAD  
Automatic merge went well; stopped before committing as requested
```

7. From your Pantheon Site Dashboard, go to the **Dev** tab and select **Code**. Make sure your site is in Git mode, and copy the Git connection information listed under Git SSH clone URL:

  ![](/source/docs/assets/images/pantheon-dashboard-git-connection-info.png)

8. From your terminal within the site directory, use the Git remote add command with an alias to make sure you know when you are moving code to or from Pantheon:

 ```bash
 git remote add pantheon <Git SSH Clone URL from Pantheon Dashboard>
 ```

  **Example:**
  ```bash
  git remote add pantheon ssh://codeserver.dev.{site-id}@codeserver.dev.{site-id}.drush.in:2222/~/repository.git site-name
  ```

  <div class="alert alert-warning" role="alert">
  <h4>Note</h4>
  Remove the site name from the end of the connection information, otherwise you will get an error and the command will fail. The final command will look like:</div>

```bash
git remote add pantheon ssh://codeserver.dev.{site-id}@codeserver.dev.{site-id}.drush.in:2222/~/repository.git
```

9. Run git add and commit to prepare the Pantheon core merge for pushing to the repository:
 ```
 git add -A
 ```
 ```
 git commit -m "Adding Pantheon core files"
 ```
10. Now pull from your Pantheon repository master branch: `git pull pantheon master`. Handle any conflicts as needed.  
11. Git push back to your Pantheon site repository: `git push pantheon master`  
12. Go to the Code tab of your Dev environment. You will see your site's pre-existing code commit history and the most recent commits adding Pantheon's core files.

![Pantheon Dashboard with Commit Messages](/source/docs/assets/images/pantheon-dashboard-git-commit-messages.png)

## Files

**Files** - anything in `sites/default/files` for Drupal or `wp-content/uploads` for WordPress. This houses a combination of uploaded content from site users, along with generated stylesheets, aggregated scripts, image styles, etc. For information on highly populated directories, see [Known Limitations](/docs/articles/sites/known-limitations/#highly-populated-directories).

Files are stored separately from the site's code. Larger file structures can fail in the Dashboard import due to sheer volume. It's best to use a utility such as an SFTP client or rsync. The biggest issue is having the transfer stopped due to connectivity issues. To handle that scenario, try this handy bash script:  

```bash
ENV='ENV'
SITE='SITEID'

read -sp "Your Pantheon Password: " PASSWORD
if [[ -z "$PASSWORD" ]]; then
echo "Woops, need password"
exit
fi

while [ 1 ]
do
sshpass -p "$PASSWORD" rsync --partial -rlvz --size-only --ipv4 --progress -e 'ssh -p 2222' $ENV.$SITE@appserver.$ENV.$SITE.drush.in:files/* ./files/
if [ "$?" = "0" ] ; then
echo "rsync completed normally"
exit
else
echo "Rsync failure. Backing off and retrying..."
sleep 180
fi
done
```
This script connects to your Pantheon site's Dev environment and starts uploading your files. If an error occurs during transfer, rather than stopping, it waits 180 seconds and picks up where it left off.  

If you are unfamiliar or uncomfortable with bash and rsync, an FTP client that supports SFTP, such as FileZilla, is a good option. Find your Dev environment's SFTP connection info and connect with your SFTP client. Navigate to `/code/sites/default/files/`. You can now start your file upload.  

## Database  

**Database** - a single .sql dump that contains the content and active state of the site's configurations.

You'll need a .sql file containing the data from the site you want to import. If you haven't done so already, make sure you remove any data from the cache tables. That will make your .sql file much smaller and your import that much quicker.


1. From the Dev environment on the site Dashboard, click **Connection Info** and copy the Database connection string. It will look similar to this:

 ```
 mysql -u pantheon -p{massive-random-pw} -h dbserver.dev.{site-id}.drush.in -P {site-port} pantheon
 ```
2. From terminal, `cd` into the directory containing your `.sql` archive. Paste the connection string and ammend it with:
`< database.sql`
Your command will now look like:

 ```
 mysql -u pantheon -p{massive-random-pw} -h dbserver.dev.{site-id}.drush.in -P {site-port} pantheon < database.sql
 ```
3. After you run the command, the .sql file is imported into your Pantheon Dev database.  

You should now have all three of the major components of your site imported into Pantheon. Clear your caches via the Pantheon Dashboard, and you are good to go.
