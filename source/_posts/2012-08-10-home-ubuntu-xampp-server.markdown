---
layout: post
title: "Home Ubuntu XAMPP Server"
date: 2012-08-10 12:00
categories: 
- server
- ubuntu
- drupal
- guide
---

## Resources

* Linux: I'm using Ubuntu 12.04 Alternate distribution, because of the option of using LVM for the partitions.
* XAMPP: Easy way of getting LAMP up and running
* Phpsysinfo: To check server resources and information (self-contained php website)
* Drupal: The actual site I will be hostin
* OpenSSH-server: To allow for remotely controlling the server
* dnsdynamic.org: The site where I register for a free domain.
* DDClient: The tool that will keep the dynamic dns updated on the domain

## Installation

If you want to install the alternate distrubution of a usb, you're best bet is using an existing Ubuntu install to create the live-usb. By using the Ubuntu USB Creator tool you will avoid any trouble that the alternate distribution has with other live usb creators in regard to components it needs to load of "cd" which in this case is USB, but for some reason doesn't get picked up by the installer.

## LVM Configuration

The computer I'm installing on has a 250GB HDD.

_Note:_ Keep /boot on a seperate primary ext4 partition, safer than to put it under LVM as well. Recommended size: 10Gb

Afterwards, create a logical volume of the remaining disk space. In the option of "use as", set it as the physical volume for LVM.

Now configure the Logical Volume manager.

* First create a volume group and point it towards the physical volume (don't forget to actually select the volume by pressing spacebar).
* The next step is to create a Logical Volume (LV) for every partition you want to have on the drive.

The one I went with for the server is the following, considering I have a 10gb /boot partition from earlier.

    * /home 20GB
    * /opt 25GB
    * /var 15GB
    * /usr 15GB
    * /tmp 5GB
    * swap 4GB
   
This leaves plenty of room to grow any partition that needs it, as this is usually easier than shrinking one.

It's safest to choose an ext4 filesystem for the partitions (except swap, which doesn't have a filesystem) but keep in mind that it's necessary to unmount a partition based on ext4 before you can resize it (there are filesystem with no such restriction like ReiserFS).

The final step is to actually mount said partitions on the Logical volumes we just defined, as well as choosing their filesystems.

## Installing XAMPP

Follow the guidelines on the XAMPP site, which boil down to this:

    sudo tar xvfz xampp-*.tar.gz -C /opt
    
To start XAMPP:

    sudo /opt/lampp/lampp start
    
and to configure the security settings:

    sudo /opt/lampp/lampp security
    

If you chose a password for XAMPP, any site in /opt/lampp/htdocs/ will be password protected (at least it seems to be every site, the documentation only mentions the /xampp folder).

Connections from outside the network will also be disabled by default.

Some extra security hardening guidelines:
  [XAMPP Security Hardening](http://robsnotebook.com/xampp-security-hardening)[robsnotebook.com]
  
## Extra security

### WebDAV

Disable WebDav if you don't need it, it may have a default password and user.

Open up the config file
    
    sudo gedit /opt/lampp/etc/extra/httpd-dav.conf
    
Change the following
    
    <Directory "/opt/lampp/uploads">
        Dav On
        
To

    <Directory "/opt/lampp/uploads">
        Dav On
        
Set a password and a user for "admin"

    sudo htdigest -c "/opt/lampp/user.passwd" DAV-upload admin
    
and enter a password twice.

### ProFTPD

Don't start the service if you don't need it (XAMPP does not have to be running for this)

    sudo /opt/lampp/lampp stopftp
    
### Default XAMPP user "lampp"

To change to a different username:

    sudo gedit /opt/lampp/lib/xampp.users
    
and change "lampp" to something else.

### Setting up site-wide login

After /opt/lampp/lampp security has been run, only the default folders were password protected. A new folder, called /test, will not be password protected.

Adding .htaccess files solves this, but is slower than defining it in httpd.conf and this can be set to work for the whole htdocs folder.

    sudo gedit /opt/lampp/etc/httpd.conf
    
There is a "<Directory>" listing there.
Change the line "AllowOverride All" to "AllowOverride None", disabling all .htaccess files in the folder.

Next, add the controls for who can access this folder
_Note:_ You can skip to group access if you want to set up group access straight away

    AuthName "User"                 # The welcome message on the login screen
    AuthType Basic                  
    AuthUserFile /opt/lampp/lib/xampp.users # Where the user:password items are found
    Require valid-user              # must be a valid user before he can log in
    
Notice that we changed 'AuthName "User"' from 'AuthName "Xampp User"' to not give away any information from the login screen.

### Setting up groups

Add a file /opt/lampp/lib/xampp.groups

defining wich users (from xampp.users) belong to what group.

    group1:user1 user2
    group2:user3
    
Next, change the controls in /opt/lampp/etc/httpd.conf

    AuthName "User"                 # The welcome message on the login screen
    AuthType Basic                  
    AuthUserFile /opt/lampp/lib/xampp.users # Where the user:password items are found
    AuthGroupFile /opt/lampp/lib/xampp.groups # Where the groups are defined
    Require group admin             # must be a valid user before he can log in
    
    # works like this:
    # Require group group1 group2       # for multiple groups
    # Require user adminuser            # You can specify additional users
    
### SSL only access to password protected folders

In this case, all folders.

Generate a certificate and put the certificate and its key in the correct folders.
Parameters provided are:

    -nodes don't encrypt private key
    -new generate a new certificate
    -x509 self sign the certificate
    -days n certify the certificate for n days
    
    sudo openssl req $@ -new -x509 -days 365 -nodes -out /opt/lampp/etc/ssl.crt/apache.crt -keyout /opt/lampp/etc/ssl.key/apache.key
    
Fill in all the fields or provide '.' to let it stay empty, all but COMMON NAME, which needs to be your domain name. If you were google, it would be wwww.google.com

Check that the permissions on apache.crt are 644 and on apache.key are 640
   
Then

    sudo gedit /opt/lampp/etc/httpd.conf
    
and add to the "<Directory>" listing of the folder you want to protect with SSL

    SSLRequireSSL
    
Then

    sudo gedit /opt/lampp/etc/extra/httpd-ssl.conf
    
change the location of ssl.cert and ssl.key.

### Redirect all traffic to https

 Do
 
    sudo /opt/lampp/etc/httpd.conf
    
 and add
 
    <IfModule mod_rewrite.c>
        RewriteEngine On

        # Redirect all traffic to https
        RewriteCond %{HTTPS} !=on
        RewriteCond %{REQUEST_URI} /
        RewriteRule ^(.*) https://%{SERVER_NAME}$1 [R,L]
    </IfModule>
    
If you want only specific folders to be redirected, change "/" to the folder path, for example "/secure".

### Linux firewall - UFW (GUFW gui)

install it and open ports 80, 443 (if you're using SSL) and 22 (if you're using SSH)

## Drupal

Installing is as easy as copying the drupal download to htdocs folder.

You have to duplicate the default.settings.php into settings.php under sites/default

You also have to change the permissions on the file and folder during installation

    chmod 666 settings.php
    chmod 776 ../default
    
After the installation you can change it back to

    chmod 640 settings.php
    chmod 755 ../default
    
apache needs write access to settings.php(change owner and group to apache server)
    
## Change XAMPP Username and group

in the httpd.conf file 
change user and group

usermod -l newname oldname
usermod -g newgroup oldgroup

(in case of errors, change /etc/user.passwd and /etc/group)

and change it in /opt/lampp/etc/my.cnf as well

restart
    
## Error on status report page Drupal

Error:

    Your system or network configuration does not allow Drupal to access web pages, resulting in reduced functionality. This could be due to your webserver configuration or PHP settings, and should be resolved in order to download information about available updates, fetch aggregator feeds, sign in via OpenID, or use other network-dependent services. If you are certain that Drupal can access web pages but you are still seeing this message, you may add $conf['drupal_http_request_fails'] = FALSE; to the bottom of your settings.php file.

This error has many causes and quite a few ways of fixing it (as well as just supressing the error) but if your hosting the server yourself behind a firewall on a router, it might be because drupal tries to access itself from the outside, which some firewalls see as an attack and block it.

To solve this, you can avoid this unnecessary detour through the firewall and just tell your server what ip is connected to the domain.

Say, your server is hosting myserver.dns.org and the internal network ip is 192.168.1.5, you would add the following entry to /etc/hosts

    192.168.1.5     myserver.dns.org
    
which tells your server to go to 192.168.1.5 when it is looking for myserver.dns.org, avoiding the firewall on the router.

Additionally

  # Modify the RewriteBase if you are using Drupal in a subdirectory or in a
  # VirtualDocumentRoot and the rewrite rules are not working properly.
  # For example if your site is at http://example.com/drupal uncomment and
  # modify the following line:
    RewriteBase /mydrupaldirectory

might be necessary after enabling clean-urls

## Drupal Clean URLS

    Note: Occasionally the test gives a false negative result. See http://drupal.org/node/1178850. To avoid this make sure you navigate to a clean URL like http://www.example.co.uk/admin/config/search/clean-urls rather than simply refreshing a non-clean URL like http://www.example.co.uk/?q=admin/config/search/clean-urls.

## Disable SSH root login

	 
    sudo /etc/ssh/sshd_config
    
and change 

    PermitRootLogin Yes
   
to

    PermitRootLogin No
    
## List all files with 777 permission and not located in folder X

     find / -type f -perm 0777 -and -not -wholename /folder_to_ignore_1\* -prune -and -not -wholename /folder_to_ignore_2\* -prune 2>/dev/null | xargs ls -lc --full-time > permission777list.txt
     
 This command will find all the files with permission 777 located under root (recursively), excet folder1 and 2 and any of their subfolders (-prune avoids going through the entire folder, just to ignore it anyway), ignore errors with 2>/dev/null. Then pass this list to xargs ls to show all the permission info (-l) and modification date(-c) with the date and time printed in human full-iso format. This fill is then saved in permission777list.txt.
 
 ## Temperature sensors - lm-sensors
 
    sudo apt-get install lm-sensors
    
    sudo sensors-detect
    
    sensors
 
## things that did not work 
 
### XAMPP PHP enabling extensions
 
extension=zip.so does not work

sudo apt-get install zlib1g-dev

wget xampp-dev-url

sudo tar xvfz xampp-linux-devel-1.8.0.tar.gz -C /opt

sudo /opt/lampp/bin/pecl install zip

won't install if version of zip is not set for version of php
 
### Nano
 
smooth scrolling makes for a very low scroll speed
    
## Owncloud server (under XAMPP)


## Installing LAMP From scratch

[Source](http://kiranjith83.blogspot.be/2009/09/hardening-linux-web-servers-lamp.html)

### Tasksel

sudo tasksel

choose LAMP-server ( leave the rest on , as it also shows what is already installed)

### MySQL

mysql_secure_installation

### PHPMyAdmin

enable PSStatus, BAT

### enable some apache mods

sudo a2enmod

### apache mod_ssl

create an .ssh folder
sudo openssl genrsa -out dev-server.key 1024

sudo openssl req -new -key dev-server.key -out dev-server.csr

sudo openssl x509 -in dev-server.csr -out dev-server.crt -req -signkey dev-server.key -days 365

### Configure password access

create an auth folder

sudo htpasswd users titan

sudo nano groups
    admin:titan
    
## Drupal migration

settings.php
.htaccess rewritebase

## Owncloud

ServerName domain.dns.info

## APC

apt-get install apache2-threaded-dev


pecl install apc

apache restart

php --ini

/etc/php5/cli/conf.d/apc.ini

manual entry of vars

pear config-set php_ini /etc/php5/cli/php.ini
pecl config-set php_ini /etc/php5/cli/php.ini

### Denyhosts won't start

sudo /usr/sbin/denyhosts

will tell you what errors it encountered during startup

### rsnapshot - dual backups

make a second config

run rsnapshot with -c /path/to/configfile

### phpsysinfo

#### PSStatus

//CommonFunctions::executeProgram("pidof", "-s ".$process, $buffer, PSI_DEBUG);
CommonFunctions::executeProgram("pgrep", "-x ".$process." | tail -1", $buffer, PSI_DEBUG);
    
### Drupal - uploadprogressbar

sudo PECL install uploadprogressbar

### DNS & vhost

http://ohonei.hubpages.com/hub/Configure-Apache2-VirtualHost-and-Bind9-on-Debian

Important to know

If blocking / allowing people, it's important to set the correct "Satisfy/require" setting
THere are 2 things that get checked, the Allow/deny statements and the Auth statements

Satisfy All will check both, if one does not check out you are automatically blocked, with Satisfy Any, either one can check out and you are allowed.

This is very important to know, for instance:

You want to allow everybody

Order Allow,Deny
Allow from all
Satisfy Any

=> Satisfy Any, because you don't want to check for authentication since you want to allow everybody, everybody validates on the "Allow from all" statement(obviously) so Auth does not get checked. If you had put Satisfy All, he would also check for Auth, which is not there, causing error 500 

You want to block everybody:

Order Deny,Allow
Deny from all
Satisfy All

=> Satisfy All, because you don't want to check Auth and everybody validates on the deny from all statement so Auth does not need to be checked.


 


  
