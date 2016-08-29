## Installation

### homebrew
```
xcode-select --install
```
```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.bash_profile
brew doctor
```

### Java
```
java -version
```

### Elasticsearch
```
brew install elasticsearch
brew info elasticsearch
```

1. Start Elasticsearch
```
elasticsearh
```

### Plugins

#### elasticsearch-head
```
/usr/local/Cellar/elasticsearch/2.3.4/libexec/bin/plugin install mobz/elasticsearch-head
```

Check here: http://localhost:9200/_plugin/head/

### Sense
Download https://chrome.google.com/webstore/detail/sense-beta/lhjgkmllcaadmopgmanpapmpjgmfcfig?hl=en

## Documentation
https://docs.google.com/presentation/d/1Ne6l-MbYqdsaFL9L9W_mzrnCYekzRc5oQ6AyWwsY4bM/edit?usp=sharing
https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html

## Index data

1. Check official Documentation, Copy/Paste the twitter example and change it to index Popety staff (Name, Skills, Nationality)

1. Check mapping rules

1. Add Age

1. Check mapping rules

## Retrieve data

> Query context
> A query clause used in query context answers the question “How well does this document match this query clause?”

> Filter context
> In filter context, a query clause answers the question “Does this document match this query clause?”

https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html

1. Query all data: match_all query

1. Query data with term 'node' in skills field. what about fuzzy instead of match ?

1. highlight the result

1. Filter by language -> Explain Mapping and analyze

## Cluster

### Create a Cluster all together

1. Stop ES

1. Open YAML configuration file:
```
brew info elasticsearch
```
1. and change the following confs:
```
cluster.name: popety
node.name: YOUR_NAME
network.host: YOUR_IP
discovery.zen.ping.unicast.hosts: ["OTHER_DEV_IP", "OTHER_DEV_IP"]
```

1. Let's play with Popety Data:
* GeoQueries search
* Aggregation
* ?
