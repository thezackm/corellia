# Overview

```shell
cat /etc/os-release  | grep "PRETTY"
PRETTY_NAME="CentOS Linux 7 (Core)"
```

## Download Oracle DB Express v18c

### Pre-Install package
`wget "https://yum.oracle.com/repo/OracleLinux/OL7/latest/x86_64/getPackage/oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm" -O oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm`

### Install Package
`wget "https://download.oracle.com/otn-pub/otn_software/db-express/oracle-database-xe-18c-1.0-1.x86_64.rpm" -O oracle-database-xe-18c-1.0-1.x86_64.rpm`

*This took about 7 minutes*

## Install Dependencies

Install JDK 11
`sudo yum install java-11-openjdk-devel`

## Install Oracle Instant Client (Basic)
### Update yum repository

`sudo wget "http://yum.oracle.com/public-yum-ol7.repo" -O /etc/yum.repos.d/public-yum-ol7.repo`

### Import GPG Key
```shell
wget "http://yum.oracle.com/RPM-GPG-KEY-oracle-ol7"
sudo rpm --import RPM-GPG-KEY-oracle-ol7
```

### Install `yum-utils`

`sudo yum install yum-utils`

### Enable Oracle InstantClient Repo

`sudo yum-config-manager --enable ol7_oracle_instantclient`

### Install oracle-instantclient18.3-sqlplus.x86_64

`sudo yum install oracle-instantclient18.3-sqlplus.x86_64`

## Install PreInstall package

`sudo rpm -ivh oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm`

This will list your uninstalled dependencies that you can then install with:

 `sudo yum install -y dependency1 dependency2 etc`

## Install Oracle Express package

`sudo rpm -ivh oracle-database-xe-18c-1.0-1.x86_64.rpm`

## Configure the Initial DB Settings

### Default Config File

`sudo cat /etc/sysconfig/oracle-xe-18c.conf`

```shell
#This is a configuration file to setup the Oracle Database. 
#It is used when running '/etc/init.d/oracle-xe-18c configure'.

# LISTENER PORT used Database listener, Leave empty for automatic port assignment
LISTENER_PORT=

# EM_EXPRESS_PORT Oracle EM Express URL port
EM_EXPRESS_PORT=5500

# Character set of the database
CHARSET=AL32UTF8

# Database file directory
# If not specified, database files are stored under Oracle base/oradata
DBFILE_DEST=

# SKIP Validations, memory, space
SKIP_VALIDATIONS=false
```

### Initialize Configuration

`sudo /etc/init.d/oracle-xe-18c configure`

This will prompt you for a password that contains 1 uppercase, 1 lowercase, and 1 digit. No special characters are allowed. 

This password will be set for the following users during the config:

```
SYS
SYSTEM
PDBADMIN
```

STDOUT during the initial config: 
```shell
Specify a password to be used for database accounts. Oracle recommends that the password entered should be at least 8 characters in length, contain at least 1 uppercase character, 1 lower case character and 1 digit [0-9]. Note that the same password will be used for SYS, SYSTEM and PDBADMIN accounts:
Confirm the password:
Configuring Oracle Listener.
Listener configuration succeeded.
Configuring Oracle Database XE.
Enter SYS user password: 
*********
Enter SYSTEM user password: 
******* 
Enter PDBADMIN User Password: 
********
Prepare for db operation
7% complete
Copying database files
29% complete
Creating and starting Oracle instance
30% complete
31% complete
34% complete
38% complete
41% complete
43% complete
Completing Database Creation
47% complete
50% complete
Creating Pluggable Databases
54% complete
71% complete
Executing Post Configuration Actions
93% complete
Running Custom Scripts
100% complete
Database creation complete. For details check the logfiles at:
 /opt/oracle/cfgtoollogs/dbca/XE.
Database Information:
Global Database Name:XE
System Identifier(SID):XE
Look at the log file "/opt/oracle/cfgtoollogs/dbca/XE/XE.log" for further details.

Connect to Oracle Database using one of the connect strings:
     Pluggable database: oracle-techops/XEPDB1
     Multitenant container database: oracle-techops
Use https://localhost:5500/em to access Oracle Enterprise Manager for Oracle Database XE
```

### Set Environmental Variables

`sudo vim /etc/environment`

```shell
ORACLE_HOME=/opt/oracle/product/18c/dbhomeXE
ORACLE_SID=XE
ORAENV_ASK=NO
PATH="/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/techops/.local/bin:/home/techops/bin:/opt/oracle/product/18c/dbhomeXE/bin"
```

Reboot to reload Environment

`sudo reboot -h now`

### Set Oracle to Autostart

Start the Oracle service

`udo systemctl start oracle-xe-18c`

Setup Autostart

```shell
sudo systemctl daemon-reload
sudo systemctl enable oracle-xe-18c
```

### Login and Validate

Login as the `SYSTEM` user, using the password you set in the initial config

`sqplus SYSTEM`

Query the Oracle DB Version to validate everything is working

`select * from V$VERSION;`

