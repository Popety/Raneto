# Set up phpMyAdmin on Digital Ocean VPS

## Create server

Create the smallest server available and don't forget to activate Private Network

## EPEL Package

Extra Packages for Enterprise Linux (EPEL) Repository will provide the phpMyAdmin package.

1. Install EPEL (change URL according to latest version)
```
sudo yum install epel-release
```

## Install a LAMP Stack

1. Install the necessary components from the default yum repositories
```
sudo yum install httpd php php-mysql
```

## Configure the LAMP Stack

1. Change the apache port in the httpd configuration file
```
vi /etc/httpd/conf/httpd.conf
```
and changing the following lines
```
Listen 80
```
by
```
Listen 3300
```
> use the port you want

2. Test everything os by starting the Web Server
```
sudo service httpd start
```

3. Start Apache by default on reboot
```
systemctl enable httpd.service
```

> opening http://your_ip_adress:3300 you should see a "Testing 123.." page

## Install phpMyAdmin

1. Install the phpMyAdmin package
```
sudo yum install phpmyadmin
```

## Configure phpMyAdmin

1. Add password to apache user (the one running apache server)
```
passwd apache
```

2. Open the phpMyAdmin mysql configuration file
```
vi /etc/phpMyAdmin/config.inc.php
```

3. Update the ip of the database
```
$cfg['Servers'][$i]['host']          = 'your_mysql_ip'; // MySQL hostname or IP address
```

4. Open the phpMyAdmin conf file
```
vi /etc/httpd/conf.d/phpMyAdmin.conf
```
Remove the "/usr/share/phpMyAdmin/setup/" section:
```
<Directory /usr/share/phpMyAdmin/setup/>
   <IfModule mod_authz_core.c>
     # Apache 2.4
     <RequireAny>
       #Require ip 127.0.0.1
       #Require ip ::1
       Require all granted
     </RequireAny>
   </IfModule>
   <IfModule !mod_authz_core.c>
     # Apache 2.2
     Order Deny,Allow
     Deny from All
     Allow from 127.0.0.1
     Allow from ::1
   </IfModule>
</Directory>
```
Update the "/usr/share/phpMyAdmin/" section as follow:
```
<Directory /usr/share/phpMyAdmin/>
   AddDefaultCharset UTF-8

   <IfModule mod_authz_core.c>
     # Apache 2.4
     <RequireAny>
       #Require ip 127.0.0.1
       #Require ip ::1
       Require all granted
     </RequireAny>
   </IfModule>
   <IfModule !mod_authz_core.c>
     # Apache 2.2
     Order Deny,Allow
     Deny from All
     Allow from 127.0.0.1
     Allow from ::1
   </IfModule>
</Directory>
```

5. Restart Apache
```
systemctl restart httpd.service
```

## Annex
https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-a-centos-6-4-vps
