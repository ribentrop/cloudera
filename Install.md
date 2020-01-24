# Before You Install
### Storage Space Planning for Cloudera Manager
### Configure Network Names
### Disabling the Firewall
### Required Privileges
Сделал установку через root 
!!! Разобраться через установку через нормальную учетку
```sh
sudo -s
sudo passwd
vi /etc/ssh/sshd_config
change “PermitRootLogin” to “yes”
change “PasswordAuthentication” to “yes”
service sshd restart
пароль: ПАРОЛЬ
```
### Ports
Просто открыл порты на CLOWD
!!! Разобраться с с корректной сетевой диаграммой


# Step 1: Configure a Repository for Cloudera Manager
```sh
sudo yum install wget
sudo wget https://archive.cloudera.com/cm6/6.3.1/redhat7/yum/cloudera-manager.repo -P /etc/yum.repos.d/
sudo rpm --import https://archive.cloudera.com/cm6/6.3.1/redhat7/yum/RPM-GPG-KEY-cloudera
```
# Step 2: Install JDK
```sh
sudo yum install oracle-j2sdk1.8 # Install JDK
```
# Step 3: Install Cloudera Manager Server
### Install Cloudera Manager Packages
On the Cloudera Manager Server host, type the following commands to install the Cloudera Manager packages.
```sh
sudo yum install cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server
```

# Step 4: Install and Configure Databases
### Installing PostgreSQL Server
```sh
sudo yum install postgresql-server
```
Notes:
- Make sure that the data directory, which by default is /var/lib/postgresql/data/, is on a partition that has sufficient free space.
### Installing the psycopg2 Python Package

If you are installing or upgrading to CDH 6 and using PostgreSQL for the Hue database, you must install psycopg2 2.5.4 or higher on all Hue hosts as follows. These examples install version 2.7.5 (2.6.2 for RHEL 6):

1. Install the python-pip package:
```sh
sudo yum install python-pip
```

2. Install psycopg2 2.7.5 using pip:
```sh
sudo pip install psycopg2==2.7.5 --ignore-installed
```
Может так же потребоваться:
```sh
sudo pip install psycopg2==2.7.5 --ignore-installed
```
### Configuring and Starting the PostgreSQL Server
By default, PostgreSQL only accepts connections on the loopback interface. You must reconfigure PostgreSQL to accept connections from the fully qualified domain names (FQDN) of the hosts hosting the services for which you are configuring databases. If you do not make these changes, the services cannot connect to and use the database on which they depend.

1. Make sure that LC_ALL is set to en_US.UTF-8 and initialize the database as follows:
```sh
echo 'LC_ALL="en_US.UTF-8"' >> /etc/locale.conf
sudo su -l postgres -c "postgresql-setup initdb"
```
2. Enable MD5 authentication. Edit pg_hba.conf, which is usually found in /var/lib/pgsql/data or /etc/postgresql/<version>/main. Add the following line:
```sh
host all all 127.0.0.1/32 md5
```
If the default pg_hba.conf file contains the following line:
```sh
host all all 127.0.0.1/32 ident
```
then the host line specifying md5 authentication shown above must be inserted before this ident line.
```sh
sudo vi /var/lib/pgsql/data/pg_hba.conf
```
ALLOW REMOTE CONNECTIONS:
```sh
sudo vi /var/lib/pgsql/data/postgresql.conf
listen_addresses = '*'
```
3. Configure the PostgreSQL server to start at boot.
```sh
sudo systemctl enable postgresql
```
Restart the PostgreSQL database:
```sh
sudo systemctl restart postgresql
```
### Creating Databases for Cloudera Software
МОЖНО ВЫПОЛНИТЬ СКРИПТ:
```sh
sudo -u postgres psql  -f sqltest.sql
```
1. Connect to PostgreSQL:
```sh
sudo -u postgres psql
```
2. Create databases for each service you are using from the below table:
```sh
CREATE ROLE <user> LOGIN PASSWORD '<password>';
CREATE DATABASE <database> OWNER <user> ENCODING 'UTF8';
```
```sh
CREATE ROLE scm LOGIN PASSWORD 'scm';
CREATE DATABASE scm OWNER scm ENCODING 'UTF8';
CREATE ROLE amon LOGIN PASSWORD 'amon';
CREATE DATABASE amon OWNER amon ENCODING 'UTF8';
CREATE ROLE rman LOGIN PASSWORD 'rman';
CREATE DATABASE rman OWNER rman ENCODING 'UTF8';
CREATE ROLE hue LOGIN PASSWORD 'hue';
CREATE DATABASE hue OWNER hue ENCODING 'UTF8';
CREATE ROLE metastore LOGIN PASSWORD 'metastore';
CREATE DATABASE metastore OWNER metastore ENCODING 'UTF8';
CREATE ROLE sentry LOGIN PASSWORD 'sentry';
CREATE DATABASE sentry OWNER sentry ENCODING 'UTF8';
CREATE ROLE nav LOGIN PASSWORD 'nav';
CREATE DATABASE nav OWNER nav ENCODING 'UTF8';
CREATE ROLE navms LOGIN PASSWORD 'navms';
CREATE DATABASE navms OWNER navms ENCODING 'UTF8';
CREATE ROLE oozie LOGIN PASSWORD 'oozie';
CREATE DATABASE oozie OWNER oozie ENCODING 'UTF8';
```
3. For PostgreSQL 8.4 and higher, set standard_conforming_strings=off for the Hive Metastore and Oozie databases:
```sh
ALTER DATABASE metastore SET standard_conforming_strings=off;
```
```sh
ALTER DATABASE oozie SET standard_conforming_strings=off;
```

# Step 5: Set up the Cloudera Manager Database
### Syntax for scm_prepare_database.sh
```sh
sudo /opt/cloudera/cm/schema/scm_prepare_database.sh [options] <databaseType> <databaseName> <databaseUser> <password>
```
### Preparing the Cloudera Manager Server Database
1. Run the scm_prepare_database.sh script on the Cloudera Manager Server host, using the database name, username, and password you created in Step 4
```sh
sudo /opt/cloudera/cm/schema/scm_prepare_database.sh postgresql scm scm scm
```
2. Remove the embedded PostgreSQL properties file (хз,что это такое...):
```sh
sudo rm /etc/cloudera-scm-server/db.mgmt.properties
```
# Step 6: Install CDH and Other Software
Start Cloudera Manager Server:
```sh
sudo systemctl start cloudera-scm-server
```
Wait several minutes for the Cloudera Manager Server to start. To observe the startup process, run the following on the Cloudera Manager Server host:
```sh
sudo tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
```
When you see this log entry, the Cloudera Manager Admin Console is ready:
```sh
INFO WebServerImpl:com.cloudera.server.cmf.WebServerImpl: Started Jetty server.
```
In a web browser, go to http://<server_host>:7180, where <server_host> is the FQDN or IP address of the host where the Cloudera Manager Server is running.
Log into Cloudera Manager Admin Console. The default credentials are:
```sh
Username: admin
Password: admin
```
# Step 6: Install CDH and Other Software
### Select Edition
Choose which edition to install:
- Essentials Edition
### Cluster Basics
Regular Cluster: 
- A Regular Cluster contains storage nodes, compute nodes, and other services such as metadata and security collocated in a single cluster.
### Select Repository
Без изменений
### Enter Login Credentials
Добавил себя в судоеры. Указал в качестве пользователя себя.
```sh
sudo  usermod -aG wheel justribentrop_gmail_com
```





New Features!

# Dillinger

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)


Markdown is a lightweight markup language based on the formatting conventions that people naturally use in email.  As [John Gruber] writes on the [Markdown site][df1]

> The overriding design goal for Markdown's
> formatting syntax is to make it as readable
> as possible. The idea is that a
> Markdown-formatted document should be
> publishable as-is, as plain text, without
> looking like it's been marked up with tags
> or formatting instructions.


* [AngularJS] - HTML enhanced for web apps!
* [Ace Editor] - awesome web-based text editor
* [markdown-it] - Markdown parser done right. Fast and easy to extend.
* [Twitter Bootstrap] - great UI boilerplate for modern web apps
* [node.js] - evented I/O for the backend
* [Express] - fast node.js network app framework [@tjholowaychuk]
* [Gulp] - the streaming build system
* [Breakdance](https://breakdance.github.io/breakdance/) - HTML to Markdown converter
* [jQuery] - duh



Dillinger is currently extended with the following plugins. Instructions on how to use them in your own application are linked below.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |


**Free Software, Hell Yeah!**

[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)


   [dill]: <https://github.com/joemccann/dillinger>
   [git-repo-url]: <https://github.com/joemccann/dillinger.git>
   [john gruber]: <http://daringfireball.net>
   [df1]: <http://daringfireball.net/projects/markdown/>
   [markdown-it]: <https://github.com/markdown-it/markdown-it>
   [Ace Editor]: <http://ace.ajax.org>
   [node.js]: <http://nodejs.org>
   [Twitter Bootstrap]: <http://twitter.github.com/bootstrap/>
   [jQuery]: <http://jquery.com>
   [@tjholowaychuk]: <http://twitter.com/tjholowaychuk>
   [express]: <http://expressjs.com>
   [AngularJS]: <http://angularjs.org>
   [Gulp]: <http://gulpjs.com>

   [PlDb]: <https://github.com/joemccann/dillinger/tree/master/plugins/dropbox/README.md>
   [PlGh]: <https://github.com/joemccann/dillinger/tree/master/plugins/github/README.md>
   [PlGd]: <https://github.com/joemccann/dillinger/tree/master/plugins/googledrive/README.md>
   [PlOd]: <https://github.com/joemccann/dillinger/tree/master/plugins/onedrive/README.md>
   [PlMe]: <https://github.com/joemccann/dillinger/tree/master/plugins/medium/README.md>
   [PlGa]: <https://github.com/RahulHP/dillinger/blob/master/plugins/googleanalytics/README.md>
