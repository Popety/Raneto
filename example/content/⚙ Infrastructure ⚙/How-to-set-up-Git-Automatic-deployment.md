* Host: DigitalOcean
* OS: CentOs
* PM2

## Create no root user

## Create a bare repository

1. Add SSH server key to Github

1. Create a repo folder
```
cd /var
mkdir repo && cd repo
```

1. Create an empty Git bare repository
```
mkdir YOUR_PROJECT.git
cd YOUR_PROJECT.git
git init --bare
```

1. Create a post-receive file in hooks directory
```
cd hooks
vi post-receive
```

1. Set up auto deployment script with PM2
```
#!/bin/sh
#
## store the arguments given to the script
read prevCommitSHA latestCommitSHA fullbranchName

branch=$(git rev-parse --symbolic --abbrev-ref $fullbranchName)

WEBROOT="/var/www"
PROJECT="web"
LOGFILE="$WEBROOT/$PROJECT/post-receive.log"
DEPLOYDIR=null
PORT=null

echo "log: $LOGFILE"

##  Record the fact that the push has been received
echo "Received Push Request at $( date +"%F %T" ) for #branch $branch"  >> $LOGFILE

echo "Checking deployment rules for project:$PROJECT, branch: $branch"

# Log the branch name
echo "---------------------------Deploy Start-------------------------------------" >> $LOGFILE

if [ $branch = "master" ]
    then
    DEPLOYDIR="$WEBROOT/$PROJECT/master"
    PORT=3100
    pm2_name=master
    #API_URL="var baseURL = 'https://dev.popety.com/api-master/api/v2/';"
fi

if [ $branch != "master" ]
    then
    DEPLOYDIR="$WEBROOT/$PROJECT/test"
    PORT=3101
    pm2_name=test
    #API_URL="var baseURL = 'https://dev.popety.com/api-test/api/v2/';"
fi

pm2 stop "${PROJECT}_${pm2_name}" >> $LOGFILE
echo "Stopped server at $( date +"%F %T" )" >> $LOGFILE

echo "Deploying to $DEPLOYDIR" >> $LOGFILE

echo " - Starting code checkout"  >> $LOGFILE
GIT_WORK_TREE="$DEPLOYDIR" git checkout -f "$branch"
echo " - Finished code checkout"  >> $LOGFILE

#echo " - Modify config file " >> $LOGFILE
#echo $API_URL > $DEPLOYDIR/public/web/js/config.js

echo " - Starting npm install" >> $LOGFILE
cd "$DEPLOYDIR"
rm -rf node_modules
npm install  >> $LOGFILE
echo " - Finished npm install"  >> $LOGFILE

echo "Restarting server using pm2 at $( date +"%F %T" )" >> $LOGFILE

PORT="$PORT" pm2 start server.js --name "${PROJECT}_${pm2_name}" >> $LOGFILE
echo "Restart completed at $( date +"%F %T" )"  >> $LOGFILE
cd - 1>>/dev/null
echo "---------------------------Deploy Complete---------------------------------" >> $LOGFILE

echo "Done. Run 'pm2 ls' on the server to see the process status."
```

1. Make the file executable
```
chmod +x post-receive
```


## Configure new remote branch on local

1. Add new Remote to your local Git
```
git remote add deploy-dev ssh://dev@dev.popety.com/var/repo/web.git
```

1. Push the new commit on master to dev
```
git push deploy-dev master
```

## Annex
https://www.digitalocean.com/community/tutorials/how-to-set-up-automatic-deployment-with-git-with-a-vps
http://javascript.tutorialhorizon.com/2014/08/17/push-to-deploy-a-nodejs-application-using-git-hooks/
