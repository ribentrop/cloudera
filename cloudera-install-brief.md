### Prerequisites
0. Install wget
```sh
yum install wget -y
```
1. The installation was made om CentOS Linux release 7.7.1908.
2. Disable Firewall for ports 7180,8888 TCP ports.
3. The installation was made under 'root'.
4. There is a passwordless 'root' access from Cloudera Manager node to other nodes.
5. The steps below were made in Cloudera Manager server.
6. Update a repository before install
```sh
yum update -y
```
7. Prerequisites for OT LAB PROXY installation:
- make sure the following utilities are configured to use proxy when running from shell script: __yum,wget,curl__
- make sure the following utilities are not using proxy for the internal lab addresses: __wget,curl__
- you had better configure NTP server in your lab environment (optional - may be throttled later in CM)
- you had better make DNS resolution for your lab environment (optional - may be throttled later in CM)
### Step 1: Configure a Repository for Cloudera Manager 
```sh
yum install wget -y
wget https://archive.cloudera.com/cm6/6.3.1/redhat7/yum/cloudera-manager.repo -P /etc/yum.repos.d/
rpm --import https://archive.cloudera.com/cm6/6.3.1/redhat7/yum/RPM-GPG-KEY-cloudera
```
### Step 2: Install JDK
```sh
yum install oracle-j2sdk1.8 -y
```
### Step 3: Install Cloudera Manager Packages
On the Cloudera Manager Server host, type the following commands to install the Cloudera Manager packages.
```sh
yum install cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server  -y
```
### Step 4: Installing PostgreSQL Server
```sh
yum install postgresql-server  -y
```
Notes:
- Make sure that the data directory, which by default is /var/lib/postgresql/data/, is on a partition that has sufficient free space.
#### Installing the psycopg2 Python Package
If you are installing or upgrading to CDH 6 and using PostgreSQL for the Hue database, you must install psycopg2 2.5.4 or higher on all Hue hosts as follows:
1. Install the python-pip package:
```sh
yum install python-pip -y
```
If "No package python-pip available" then:
```sh
yum --enablerepo=extras install epel-release
```
2. Install psycopg2 2.7.5 using pip:
```sh
pip install psycopg2==2.7.5 --ignore-installed 
```
If message like this is shown:
```sh
You are using pip version 8.1.2, however version 20.0.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
```
Then:
```sh
pip install --upgrade pip
```

#### Configuring and Starting the PostgreSQL Server
By default, PostgreSQL only accepts connections on the loopback interface. You must reconfigure PostgreSQL to accept connections from the fully qualified domain names (FQDN) of the hosts hosting the services for which you are configuring databases. If you do not make these changes, the services cannot connect to and use the database on which they depend.
1. Make sure that LC_ALL is set to en_US.UTF-8 and initialize the database as follows:
```sh
echo 'LC_ALL="en_US.UTF-8"' >> /etc/locale.conf
sudo su -l postgres -c "postgresql-setup initdb"
```
The output should be like this:
```sh
Initializing database ... OK
```
2. Enable MD5 authentication. 
Edit pg_hba.conf.
```sh
sudo vi /var/lib/pgsql/data/pg_hba.conf
```
It can usually be found in /var/lib/pgsql/data or /etc/postgresql/<version>/main. Add the following line:
```sh
host all all 0.0.0.0/0 md5
```
If the default pg_hba.conf file contains the following line:
```sh
host all all 127.0.0.1/32 ident
```
then the host line specifying md5 authentication shown above must be inserted before this ident line.

Allow remote connections:
```sh
sudo vi /var/lib/pgsql/data/postgresql.conf
listen_addresses = '*'
```


3. Configure the PostgreSQL server to start at boot and restart the PostgreSQL database.
```sh
systemctl enable postgresql
systemctl restart postgresql
```
#### Creating Databases for Cloudera Software
1. Connect to PostgreSQL (is is not allowed to login as a root so use sudo) :
```sh
sudo -u postgres psql
```
2. Create databases for each service you are using from the below table:
Example:
```sh
CREATE ROLE <user> LOGIN PASSWORD '<password>';
CREATE DATABASE <database> OWNER <user> ENCODING 'UTF8';
```
Creating databases and users:
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
You can execute a sql script  the following way
```sh
[root@localhost ~]# sudo -u postgres psql
postgres=# \i /tmp/clousera-users-roles.sql;
```
3. For PostgreSQL 8.4 and higher, set standard_conforming_strings=off for the Hive Metastore and Oozie databases:
```sh
ALTER DATABASE metastore SET standard_conforming_strings=off;
ALTER DATABASE oozie SET standard_conforming_strings=off;
```
### Step 5: Set up the Cloudera Manager Database
### Preparing the Cloudera Manager Server Database
1. Run the scm_prepare_database.sh script on the Cloudera Manager Server host, using the database name, username, and password you created in Step 4
```sh
/opt/cloudera/cm/schema/scm_prepare_database.sh postgresql scm scm scm
```
The output is:
```sh
All done, your SCM database is configured correctly!
```
2. Remove the embedded PostgreSQL properties file:
```sh
rm /etc/cloudera-scm-server/db.mgmt.properties
```
### Step 6: Install CDH and Other Software
Start Cloudera Manager Server:
```sh
systemctl start cloudera-scm-server
```
Wait several minutes for the Cloudera Manager Server to start. To observe the startup process, run the following on the Cloudera Manager Server host:
```sh
tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
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
