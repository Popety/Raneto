/*
Title: Install a nodeJS application production ready on a server
Sort: 2
*/

* Host: DigitalOcean
* OS: CentOs
* PM2

## Create server

Create the smallest server available and don't forget to activate Private Network.
If smallest server not enough, upgrade progressively until having good performance

## Set Up firewall

The app must be reachable only by our load balancer and through VPN.
See the SECURITY_POPETY.md document

## Install nodeJS

1. Install Git, NodeJS and npm
> we use Node.js and npm packages from the EPEL repository

```
yum update -y
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -Uvh epel-release-latest-7.noarch.rpm
yum install -y git nodejs npm
```

2. Use the n package to easily manage node version and install latest lts node version
```
npm install -g n
n lts
```

## Install PM2

PM2 is a process manager for Node.js applications. PM2 provides an easy way to manage and daemonize applications (run them in the background as a service).

1. Install PM2:
```
npm install -g pm2
```
> The -g option tells npm to install the module globally, so that it's available system-wide.

## Install ImageMagick (Optional)

We use node-imagemagick-native package that require ImageMagick to be installed on the server.

1. Install ImageMagick with headers
```
sudo yum install ImageMagick-c++ ImageMagick-c++-devel
```

## Install Redis (Optional)

1. Install Redis:
```
yum install redis
```

1. Start Redis as a service and allow to start at reboot
 ```
 sudo systemctl start redis.service
 sudo systemctl enable redis.service
 ```

## Setup environment variables

We need to set up the NODE_ENV environment variable to production

1. Add NODE_ENV
```
echo 'export NODE_ENV=prod' > /etc/profile.d/nodeenv.sh
```

## Create web user

Because it is insecure to run your application as root, we will create a web user for our system.

1. Create the user
```
useradd -mrU web
```

## Adding the nodeJS application

Now that we’ve added node and our user we can move on to adding our node app:

1. Create a folder for the app
```
mkdir /var/www
```

2. Set the owner to web (the new created user)
```
chown web /var/www
```

3. Set the group to web
```
chgrp web /var/www
```

4. cd into it
```
cd /var/www/
```

5. As the web user
```
su web
```

6. Clone your app repo
```
git clone https://github.com/XXXXXX/XXXXXX
```

7. Install all required packages
```
cd name_of_your_repo
npm install
```

8. Run the app to test it
```
npm start
```

## Manage application with PM2

1. Run the startup subcommand as root to generates and configures a startup script to launch PM2 and its managed processes on server boots.
```
sudo pm2 startup systemd -u web
```

## Annex
http://blog.carbonfive.com/2014/06/02/node-js-in-production/
https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-16-04
