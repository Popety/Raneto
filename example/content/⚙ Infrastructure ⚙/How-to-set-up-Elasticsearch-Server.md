* Host: DigitalOcean
* OS: CentOs

## Create server

Don't forget to activate Private Network. 8GB is good.

## Set Up firewall

The app must be reachable only by our API and through VPN.
See the SECURITY_POPETY.md document

## Create elastic user

Because it is insecure to run your application as root, we will create a elastic user for our system.

1. Create the user
```
useradd -mrU elasticsearch
```

2. Create data and logs file for ES:
```
mkdir /var/elasticsearch
mkdir /var/elasticsearch/data
mkdir /var/elasticsearch/logs
```

3. Give right to ES user to write on them:
```
chgrp elasticsearch /var/elasticsearch/data/
chgrp elasticsearch /var/elasticsearch/logs/
chown elasticsearch /var/elasticsearch/data/
chown elasticsearch /var/elasticsearch/logs/
```

## Install Java

1. Install the last openJDK
```
yum update -y
yum install java-1.8.0-openjdk.x86_64
```

2. Check the Java installation
```
java -version
```

## Install Elasticsearch

1. Download the latest Elasticsearch (Check the URL on their website)

```
wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/rpm/elasticsearch/2.3.5/elasticsearch-2.3.5.rpm
```

2. Install ES using rpm
```
sudo rpm -ivh elasticsearch-2.3.5.rpm
```
> This results in Elasticsearch being installed in ```/usr/share/elasticsearch/``` with its configuration files placed in ```/etc/elasticsearch``` and its init script added in ```/etc/init.d/elasticsearch```

3. Enable Elasticsearch to start and stop automatically with the server
```
sudo systemctl enable elasticsearch.service
```

## Setup environment variables

We need to set up the NODE_ENV environment variable to production

1. Add ES_HEAP_SIZE set to half of your RAM memory (example below is for a 8GB server)
```
echo 'export ES_HEAP_SIZE=4g' > /etc/profile.d/esenv.sh
```

## Configure elasticsearch

1. Open the YAML configuration file
```
vi /etc/elasticsearch/elasticsearch.yml
```

2. Uncomment and modify the following fields (for a single ES node)
```
node.name: zeus
cluster.name: popety
bootstrap.mlockall: true
path.data: /var/elasticsearch/data
path.logs: /var/elasticsearch/logs
```

4. Start ES
```
sudo service elasticsearch restart
```

Enjoy :)

## Annex
https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-elasticsearch-on-centos-7
