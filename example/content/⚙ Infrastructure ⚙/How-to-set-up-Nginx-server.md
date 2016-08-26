/*
Title: Set up NGINX Load balancer on Digital Ocean
Sort: 2
*/

## Create server

Create the smallest server available and don't forget to activate Private Network

## EPEL Package

Extra Packages for Enterprise Linux (EPEL) Repository will provide the Nginx package.

1. Get the latest version of EPEL on https://fedoraproject.org/wiki/EPEL

2. Install EPEL (change URL according to latest version)
```sh
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```
```sh
rpm -Uvh epel-release-latest-7.noarch.rpm
```

## Install Nginx

1. Install nginx
```sh
yum install nginx
```

## Configure nginx

1. Open the conf file
```
vi /etc/nginx/conf.d/default.conf
```

2. Add the upstream module to the top of the configuration file. The name backend can be replaced with a name of your choosing. All three backend servers are defined by their IP addresses.

```
upstream name_of_web_server  {
   server ip_of_web_server:port_of_web_server  weight=1;
}
server {
    listen       80;
    server_name  localhost;

    access_log  /var/log/nginx/popety.access.log  main;

    location / {
        proxy_pass http://name_of_web_server/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header HTTP_AUTHORIZATION $http_authorization;
    }
}

3. Remove the server part in nginx.conf configuration
```
vi /etc/nginx/nginx.conf
```

4. Restart Nginx
```
systemctl restart nginx
```

5. Make it start at reboot
```
systemctl enable nginx
```

> If something wrong you can check the syntax of your conf file: nginx -t -c /etc/nginx/nginx.conf

## Set up firewall

> See and run the script popety_lb_firewall and skip this section

1. If not, flush the current rules and set the default to ACCEPT:
```
sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -F
```

2. blocking null packets.
```
iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
```

3. Reject is a syn-flood attack.
```
iptables -A INPUT -p tcp ! --syn -m state --state NEW -j DROP
```

4. Drop XMAS packets, also a recon packet.
```
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP
```

5. Allow VPN to SSH via its private interface to private-VPS:
```
sudo iptables -A INPUT -p tcp -s vpn_private_IP --dport 22 -i eth1 -j ACCEPT
```

6. Allow loopback traffic on your server. This allows your server to use 127.0.0.1 or localhost:
```
sudo iptables -A INPUT -i lo -j ACCEPT
```

7. Allow public and private traffic that is initiated from your server. This will allow your server to access the Internet to do things like download updates or software:
```
sudo iptables -I INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

8. Allow web server traffic on public interface:
```
iptables -A INPUT -i eth1 -p tcp -m tcp --dport 80 -j ACCEPT
iptables -A INPUT -i eth1 -p tcp -m tcp --dport 443 -j ACCEPT
```

8. Drop INPUT and FORWARD chains by default. Note that we are leaving OUTPUT's default as ACCEPT, as we trust the servers on our private network:
```
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
```

## Annex
https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-16-04
