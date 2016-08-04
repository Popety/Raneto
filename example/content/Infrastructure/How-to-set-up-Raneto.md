# Set up Raneto on Digital Ocean

## Create server

Create the smallest server available and don't forget to activate Private Network

## Requirements

Node.js v4.0.0 (or later) must be installed
> See install nodeJS documentation

## Create web user

Because it is insecure to run your application as root, we will create a renato user

1. Create the user
```
useradd -mrU renato
```

2. Set a password to user
```
passwd renato
```

## Install Raneto

1. Create a folder for the app
```
mkdir /var/www/renato
```

2. Set the owner to renato (the new created user)
```
chown renato /var/www/renato
```

3. Set the group to web
```
chgrp renato /var/www/renato
```

4. cd into it
```
cd /var/www/renato
```

5. As the web user
```
su renato
```

6. Clone your app repo
```
git clone https://github.com/Popety/Raneto
```

7. Install all required packages
```
cd renato
npm install
```

8. Change the default port of Renato
```
echo 'export PORT=3400' > /etc/profile.d/renatoenv.sh
```

9. Run the app to test it
```
npm start
```

## Manage Raneto with PM2

1. start renato with pm2.
```
su renato
cd /var/www/renato/Raneto
pm2 start example/server.js
pm2 save
```

## Add authentication



## Annex
http://docs.raneto.com/install/requirements
