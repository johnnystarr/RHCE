###System configuration and management
- Configure IPv6 addresses and perform basic IPv6 troubleshooting
- Route IP traffic and create static routes
- Use firewalld and associated mechanisms such as rich rules, zones and custom rules, to implement packet filtering and configure network address translation (NAT)
- Configure a system to authenticate using Kerberos
- Configure a system as either an iSCSI target or initiator that persistently mounts an iSCSI target
- Produce and deliver reports on system utilization (processor, memory, disk, and network)
- Use shell scripting to automate system maintenance tasks

###Network services

- Use network teaming or bonding to configure aggregated network links between two Red Hat Enterprise Linux systems
```
yum -y install teamd
# nmcli con show
NAME                UUID                                  TYPE            DEVICE
Wired connection 1  f32cfcb7-3567-4313-9cf3-bdd87010c7a2  802-3-ethernet  eth1  
System eth0         257e9416-b420-4218-b1eb-f14302f20941  802-3-ethernet  eth0  
# nmcli con del f32cfcb7-3567-4313-9cf3-bdd87010c7a2
# nmcli con del 257e9416-b420-4218-b1eb-f14302f20941


####Network services are an important subset of the exam objectives. RHCE candidates should be capable of meeting the following objectives for each of the network services listed below:

- Install the packages needed to provide the service
- Configure SELinux to support the service
- Use SELinux port labeling to allow services to use non-standard ports
- Configure the service to start when the system is booted
- Configure the service for basic operation
- Configure host-based and user-based security for the service


###Scripting
- Create a script to automate system administration
- `touch get-shell.sh`
```
#!/bin/bash
if [ $# -eq 0 ]; then
	username=$USER
else
	username=$1
fi

userinfo=$(getent passwd $username)

if [ $? -ne 0 ]; then
echo "Error: cannot retrieve info for the user $username"
exit 2
fi

usershell=$(echo $userinfo | cut -f 7 -d ":")

echo "$username's shell is $usershell"
exit 0
```
- `chmod +x get-shell.sh` 
- `./get-shell.sh username`

###HTTP/HTTPS
- `vi /etc/httpd/sites-available/default.conf` Configure a virtual host
```
<VirtualHost *:80>
  ProxyPreserveHost On
  ProxyRequests Off
  ServerName mysite.com
  ServerAlias www.mysite.com
</VirtualHost>
```

- Configure access restrictions on directories
- Deploy a basic CGI application
- Configure group-managed content
- Configure TLS security

###DNS
- Configure a caching-only name server
- Troubleshoot DNS client issues

###NFS
- Provide network shares to specific clients
- Provide network shares suitable for group collaboration
- Use Kerberos to control access to NFS network shares

###SMB
- Provide network shares to specific clients
- Provide network shares suitable for group collaboration

###SMTP
- Configure a system to forward all email to a central mail server

###SSH
- Configure key-based authentication
- Configure additional options described in documentation

###NTP
- Synchronize time using other NTP peers

###Database services
- Install and configure MariaDB
- `yum install -y mariadb-client mariadb-server` 

- Backup and restore a database
```
# dump / backup
$ mariadbdump > mydb.sql
# restore
$ mariadbrestore < mysql.sql
```

- Create a simple database schema

```
mariadb -u username -p
Enter password:

> CREATE DATABASE mydb;
> USE mydb;

> CREATE TABLE movies (
  movie_id INT AUTO_INCREMENT PRIMARY KEY,
  movie_name VARCHAR(255),
  movie_type TINYINT
);
```
- Perform simple SQL queries against a database

```
# Insert Queries
> INSERT INTO movies (movie_name, movie_type) VALUES ('Star Wars', 1);
> INSERT INTO movies (movie_name, movie_type) VALUES ('Star Trek', 1);
> INSERT INTO movies (movie_name, movie_type) VALUES ('Lord Of The Rings', 2);

# Select Queries
> SELECT * FROM movies;
> SELECT * FROM movies WHERE movie_type = 1;
> SELECT * FROM movies WHERE move_name LIKE 'star wars';
```
