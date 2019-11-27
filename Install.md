
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
3. Configure the PostgreSQL server to start at boot.
```sh
sudo systemctl enable postgresql
```
Restart the PostgreSQL database:
```sh
sudo systemctl restart postgresql
```
### Creating Databases for Cloudera Software
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




