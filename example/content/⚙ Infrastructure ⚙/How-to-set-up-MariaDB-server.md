# Set up MariaDB on Digital Ocean

## Create server

Create a CentOS server available and don't forget to activate Private Network

## Install MariaDB

1. Get the latest MariaDB version by creating a new file
 ```
 vi /etc/yum.repos.d/MariaDB.repo
 ```
 with
 ```
 # MariaDB 10.1 CentOS repository list - created 2016-08-08 04:26 UTC
 # http://downloads.mariadb.org/mariadb/repositories/
 [mariadb]
 name = MariaDB
 baseurl = http://yum.mariadb.org/10.1/centos7-amd64
 gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
 gpgcheck=1
 ```

1. Install mariadb
 ```
 sudo yum update
 sudo yum install mariadb-server mariadb-client
 ```

1. Start MariaDB as a service and allow to start at reboot
 ```
 sudo systemctl start mariadb.service
 sudo systemctl enable mariadb.service
 ```

## Configure MariaDB

1. Secure the database (yes for all)
 ```
 mysql_secure_installation
 ```

1. Backup the original conf file
 ```
 cp /etc/my.cnf /etc/my.bak
 ```

1. Open my.cnf [mysqld]
 ```
 vi /etc/my.cnf
 ```
and add the following line in (this is for a 8GB memory server):
 ```
#
# The MySQL server
#
[mysqld]

# The maximum amount of concurrent sessions the MySQL server will
# allow. One of these connections will be reserved for a user with
# SUPER privileges to allow the administrator to login even if the
# connection limit has been reached.
max_connections = 100

# The maximum size of a query packet the server can handle as well as
# maximum query size server can process (Important when working with
# large BLOBs).  enlarged dynamically, for each connection.
max_allowed_packet = 16M

# *** Slow Queries Specific options ***

# Log slow queries. Slow queries are queries which take more than the
# amount of time defined in "long_query_time" or which do not use
# indexes well, if log_short_format is not enabled. It is normally good idea
# to have this turned on if you frequently add new queries to the
# system.
slow_query_log

# All queries taking more than this amount of time (in seconds) will be
# trated as slow. Do not use "1" as a value here, as this will result in
# even very fast queries being logged from time to time (as MySQL
# currently measures time with second accuracy only).
long_query_time = 2

# *** Cache Specific options ***

# Query cache is used to cache SELECT results and later return them
# without actual executing the same query once again. Having the query
# cache enabled may result in significant speed improvements, if your
# have a lot of identical queries and rarely changing tables. See the
# "Qcache_lowmem_prunes" status variable to check if the current value
# is high enough for your load.
# Note: In case your tables change very often or if your queries are
# textually different every time, the query cache may result in a
# slowdown instead of a performance improvement.
query_cache_size = 64M

# Only cache result sets that are smaller than this limit. This is to
# protect the query cache of a very large result set overwriting all
# other query results.
query_cache_limit = 2M

# Query cache type is used to define the type of cache activated
# 2 means only for cacheable queries that begin with SELECT SQL_CACHE
query_cache_type=2

# *** INNODB Specific options ***

# Table type which is used by default when creating new tables, if not
# specified differently during the CREATE TABLE statement.
default-storage-engine = INNODB

# InnoDB, unlike MyISAM, uses a buffer pool to cache both indexes and
# row data. The bigger you set this the less disk I/O is needed to
# access data in tables. On a dedicated database server you may set this
# parameter up to 80% of the machine physical memory size. Do not set it
# too large, though, because competition of the physical memory may
# cause paging in the operating system.  Note that on 32bit systems you
# might be limited to 2-3.5G of user level memory per process, so do not
# set it too high.
innodb_buffer_pool_size = 6G

# Size of each log file in a log group. You should set the combined size
# of log files to about 25%-100% of your buffer pool size to avoid
# unneeded buffer pool flush activity on log file overwrite. However,
# note that a larger logfile size will increase the time needed for the
# recovery process.
innodb_log_file_size = 512M

 ```

1. Restart MariaDB
 ```
 sudo systemctl restart mariadb.service
 ```

## Create Users and Schema

1. Connect to MySQL
 ```
 mysql -u root -p
 ```
 1. Create database
 ```
 CREATE DATABASE popety;
 ```
1. Create user
 ```
 CREATE USER 'popety'@'%' IDENTIFIED BY 'USER_PASSWORD';
 ```
1. Give the user full privileges for your new database
 ```
 GRANT ALL PRIVILEGES ON popety.* to popety@'%';
 ```

1. Flush the privileges to make the change take effect.
 ```
 FLUSH PRIVILEGES;
 ```

1. Verify that the privileges were set
 ```
 SHOW GRANTS FOR 'popety'@'%';
 ```

## Annex
https://support.rackspace.com/how-to/installing-mysql-server-on-centos/
https://www.percona.com/blog/2014/01/28/10-mysql-performance-tuning-settings-after-installation/
https://downloads.mariadb.org/mariadb/repositories/
