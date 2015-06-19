 
Here is a rundown of the basic commands we're using:
$ getfacl /var/www
Get current ACL's for the given directory or file

$ setfacl -R -m u:johndoe:rwx /var/www
setfacl - Set ACL
-R - Recursive down into files and directories
-m - Modifying ACL's (vs removing them)
u:johndoe:rwx - The user johndoe will get rwx permissions
/var/www - Give these permissions to the /var/www directory (and sub files/dirs, since this is a recursive operation via the -R flag)

$ setfacl -R -m g:www-data:rwx /var/www
The same as above, except:
g:www-data:rwx - Allow the group www-data to rwx the /var/www directory

$ setfacl -x g:www-data /var/www
-x - Remove ACL's defined for g:www-data at location /var/www

 
Create the users we'll use:
# Create user joel
root@server $ adduser joel  
# Give joel the ability to use "sudo"
root@server $ usermod -a -G sudo joel

# Create user bob
root@server $ adduser bob
# Bob, a user who can deploy web sites,
# is part of group www-data, the same group as
# our website files
root@server $ usermod -a -G www-data bob


We'll ensure files in our web root are of group "www-data". This is not necessary for ACL permissions, but we so do to keep things consistent.
# Ensure the site files have user/group "www-data"
# This is not necessary for ACL's
root@server $ chown -R www-data:www-data /var/www


Start using some ACL basics. Here we give a user permissiont to read/write/execute files and directories using ACLs instead of the classic Linux permissions.
# Check out the ACL's set by default
# These are separate from the usual user/group permissions
root@server $ getfacl /var/www

# Give use Joel the ability to rwx web files at
#   directory /var/www
# Technically he wouldn't *need* this,
# since he could use his sudo abilities
root@server $ setfacl -R -m u:joel:rwx /var/www

# Above we set ACL for existing files/dirs
# Here we will recursively (-R) set the
# defaults (-d flag) for future files/dirs
root@server $ setfacl -Rd -m u:joel:rwx /var/www

# Check new permissions added
# (current dir/files and defaults)
root@server $ getfacl /var/www


The previous two commands can be combined to set the defaults and current permissions in one shot: setfacl -R -m u:joel:rwx,d:u:joel:rwx /var/www
Next we give group-based permissions via ACL's to the web files. This is (arguably) more useful, a we can then give any user a secondary group (www-data in this case) to allow them to edit the web files, despite what the owner or group of those files are.
# Add group-based permissions, instead of user-specific
# This allows anyone in group "www-data" (like bob) to
# rwx files in /var/www
# We're setting the defaults here
root@server $ setfacl -R -m g:www-data:rwx /var/www

# Recursively (-R) set the defaults (-d)
for future files/dirs as well
root@server $ setfacl -Rd -m g:www-data:rwx /var/www

# View changes
root@server $ getfacl /var/www


To reiterate: These files/dirs don't need group "www-data" to be editable by bob. ANY USER part of group "www-data" can now edit files/dirs in directory /var/www!
Here are some things worth noting on using ACLs:
# Note to keep setfacl flags/options in two groups
# to avoid errors:
root@server $ setfacl
> Usage: setfacl [-bkndRLP] { -m|-M|-x|-X ... } file ...


Now test our new settings to see how they behave:
# Some testing:
# Joel creates a dir
joel@server $ mkdir -p /var/www/site/styles
# Ensure bob can write to that location:
bob@server $ touch /var/www/site/styles/styles.css

# See the ACL's on new dir - they are inherited from parent dir
# Thanks to ACL defaults set
root@server $ getfacl /var/www/site/styles


# Just to be "clean", make site files all
# part of group www-data. Again, this is
# unnecessary for ACL permissions
root@server $ chgrp -R www-data /var/www

# Set the gid (group id) used for new dirs/files
# in the /var/www directory - new ones will have
# group "www-data" just like their parent directory
#
# This technically isn't necessary, as ACL gorup permissions
# for group "www-data" will function based on the user's assigned
# group rather than the group that is asssigned to the site files
# ...But this is handy to know still
root@server $ chmod -R g+s /var/www

# Laslty, check set permissions
# Note the "s" where you usually see "x"
# in the group permissions
bob@server $ ls -lah /var/www
