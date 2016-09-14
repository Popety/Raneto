## Introduction

Let's Encrypt is a Certificate Authority (CA) that provides an easy way to obtain and install free TLS/SSL certificates, thereby enabling encrypted HTTPS on web servers.
We will create a SSL certificate to use it with Nginx and configure it to automatically renew.

* Host: DigitalOcean
* OS: CentOs
* Nginx

## Create a New Sudo User

We will use this user to set up HTTPS

1. Log in to your server as the root user.

1. Create a new user (change username by the one you want)
```
adduser username
```

1. Use the passwd command to update the new user's password.
```
passwd username
```

1. Use the usermod command to add the user to the wheel group.
```
usermod -aG wheel username
```

> By default, on CentOS, members of the wheel group have sudo privileges.

## Install certbot

1. Install the git and bc packages
```
sudo yum install epel-release
sudo yum install certbot
```

## Generate certificate

1. Create the following directory
```
/var/www/letsencrypt
```

1. Add to your Nginx server conf
```
location /.well-known/acme-challenge {
  root /var/www/letsencrypt;
}
```

1. Reload Nginx
```
systemctl reload nginx
```

1. Use the certonly command to obtain your certificate.
```
certbot certonly
```

* Use the admin address
* Set Webroot as /var/www/letsencrypt

## Set up automating renewal of certificates

1. Test automatic renewal for your certificates by running this command:
```
certbot renew --dry-run
```

1. If everything ok edit the crontab to create a new job
```
sudo crontab -e
```

1. Include the following content, all in one line to run the command twice a day (at 3AM and 15PM)
```
30 3,15 * * * certbot renew --quiet  >> /var/log/certbot-renew.log
```


## Generate Strong Diffie-Hellman Group

1. To further increase security, you should also generate a strong Diffie-Hellman group. To generate a 2048-bit group:
```
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

## Configure TLS/SSL on Web Server (Nginx)

1. Modify the Nginx configuration
```
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        server_name _;
        return 301 https://$host$request_uri;
}

server {
        listen 443 ssl;

        server_name YOUR_DOMAIN;

        ssl_certificate /etc/letsencrypt/live/YOUR_DOMAIN/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/YOUR_DOMAIN/privkey.pem;

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_dhparam /etc/ssl/certs/dhparam.pem;
        ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
        ssl_session_timeout 1d;
        ssl_session_cache shared:SSL:50m;
        ssl_stapling on;
        ssl_stapling_verify on;
        add_header Strict-Transport-Security max-age=15768000;

        location /.well-known/acme-challenge {
          root /var/www/letsencrypt;
        }
}
```

1. Restart Nginx
```
systemctl reload nginx
```

> You can check the server configuration with Qualys SSL
> https://www.ssllabs.com/ssltest/analyze.html?d=YOUR_DOMAIN


https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-centos-7
https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-centos-quickstart
