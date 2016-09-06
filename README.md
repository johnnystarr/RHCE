###System configuration and management
- Configure IPv6 addresses and perform basic IPv6 troubleshooting
- Route IP traffic and create static routes
```
#nmcli connection modify eth0 +ipv4.routes "192.168.122.0/24 10.10.10.1"
```
- Use firewalld and associated mechanisms such as rich rules, zones and custom rules, to implement packet filtering and configure network address translation (NAT)
- Configure a system to authenticate using Kerberos
- Configure a system as either an iSCSI target or initiator that persistently mounts an iSCSI target
- Produce and deliver reports on system utilization (processor, memory, disk, and network)
- Use shell scripting to automate system maintenance tasks

###Network services

- Use network teaming or bonding to configure aggregated network links between two Red Hat Enterprise Linux systems
```
# yum -y install teamd
# nmcli con show
NAME                UUID                                  TYPE            DEVICE
Wired connection 1  f32cfcb7-3567-4313-9cf3-bdd87010c7a2  802-3-ethernet  eth1  
System eth0         257e9416-b420-4218-b1eb-f14302f20941  802-3-ethernet  eth0  
# nmcli con del f32cfcb7-3567-4313-9cf3-bdd87010c7a2
# nmcli con del 257e9416-b420-4218-b1eb-f14302f20941
# nmcli con add type team con-name myteam0 ifname team0 config '{ "runner": {"name": "loadbalance"}}'
 team0 config '{ "runner": {"name": "loadbalance"}}'
[10655.288431] IPv6: ADDRCONF(NETDEV_UP): team0: link is not ready
[10655.306955] team0: Mode changed to "loadbalance"
Connection 'myteam0' (ab0a5f7b-2547-4d4f-8fc8-834030839fc1) successfully added.
```
Note1: If you don’t specify **con-name myteam0**, the teaming interface will be named team-team0.
Note2: Examples of configuration are available in the /usr/share/doc/teamd-*/example_configs. You can also get some examples through man teamd.conf.

Now, the file **/etc/sysconfig/network-scripts/ifcfg-myteam0** contains the main following lines:
```
DEVICE=team0
TEAM_CONFIG="{ \"runner\": {\"name\": \"loadbalance\"}}"
DEVICETYPE=Team
NAME=myteam0
ONBOOT=yes
```
Add an **IPv4** configuration:
In RHEL **7.0**:
```
nmcli con mod myteam0 ipv4.addresses "192.168.1.10/24 192.168.1.1"
nmcli con mod myteam0 ipv4.method manual
```
From **RHEL 7.1** on
```
nmcli con mod myteam0 ipv4.addresses 192.168.1.10/24
nmcli con mod myteam0 ipv4.gateway 192.168.1.1
nmcli con mod myteam0 ipv4.method manual
```
Note: If you don’t specify any IP configuration, both VM will get their ip address and gateway through DHCP by default.

Add the eth0 interface to the teaming interface:
```
nmcli con add type team-slave con-name team0-slave0 ifname eth0 master team0
[10707.777803] team0: Port device eth0 added
[10707.779146] IPv6: ADDRCONF(NETDEV_CHANGE): team0: link becomes ready
Connection 'team0-slave0' (a9a5b612-aad6-48b0-a097-88db35c898d3) successfully added.
```
Note1: If you don’t specify **con-name team0-slave0**, the teaming slave interface will be named **team-slave-eth0**.
Note2: The file **/etc/sysconfig/network-scripts/ifcfg-team0-slave0** has been created with the following main lines:
```
NAME=team0-slave0
DEVICE=eth0
ONBOOT=yes
TEAM_MASTER=team0
DEVICETYPE=TeamPort
```
Add the **eth1** interface to the teaming interface:
```
# nmcli con add type team-slave con-name team0-slave1 ifname eth1 master team0
[10750.419419] team0: Port device eth1 added
Connection 'team0-slave1' (e468dce3-a032-4088-8173-e7bee1bd4ad5) successfully added.
```
Note1: If you don’t specify **con-name team0-slave1**, the teaming slave interface will be named **team-slave-eth1**.
Note2: The file **/etc/sysconfig/network-scripts/ifcfg-team0-slave1** has been created with the following main lines:
```
NAME=team0-slave1
DEVICE=eth1
ONBOOT=yes
TEAM_MASTER=team0
DEVICETYPE=TeamPort
```
Activate the teaming interface:
```
nmcli con up myteam0
[10818.800169] team0: Port device eth1 removed
[10818.803399] team0: Port device eth0 removed
[10818.939884] team0: Port device eth1 added
[10818.941069] IPv6: ADDRCONF(NETDEV_CHANGE): team0: link becomes ready
[10818.971887] team0: Port device eth0 added
[10819.932168] IPv6: team0: IPv6 duplicate address fe80::5054:ff:fe3f:860a detected!
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/32)
```
Check the configuration:
```
# nmcli con show
NAME          UUID                                  TYPE            DEVICE
team0-slave0  a9a5b612-aad6-48b0-a097-88db35c898d3  802-3-ethernet  eth0
myteam0       ab0a5f7b-2547-4d4f-8fc8-834030839fc1  team            team0
team0-slave1  e468dce3-a032-4088-8173-e7bee1bd4ad5  802-3-ethernet  eth1
```
You can also use the **teamdctl** command to check the configuration state:
```
# teamdctl team0 state
setup:
  runner: loadbalance
ports:
  eth0
    link watches:
      link summary: up
      instance[link_watch_0]:
        name: ethtool
        link: up
  eth1
    link watches:
      link summary: up
      instance[link_watch_0]:
        name: ethtool
        link: up
```
Or to dump the configuration:
```
# teamdctl team0 config dump
{
"device": "team0",
"ports": {
"eth0": {
"link_watch": {
"name": "ethtool"
}
},
"eth1": {
"link_watch": {
"name": "ethtool"
}
}
},
"runner": {
"name": "loadbalance",
"tx_hash": [
"eth",
"ipv4",
"ipv6"
]
}
}
```
You can also get the ports status with the **teamnl** command:
```
# teamnl team0 ports
 2: eth0: up 0Mbit HD
 3: eth1: up 0Mbit HD
```
In addition, you can directly change the content of the files in the /etc/sysconfig/network-scripts directory but you need to apply the following command afterwards:

```
# nmcli con reload
```


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
