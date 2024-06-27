### Install Syslog-ng on Linux (RHEL-9)
Login as root user
```bash
sudo su
```

```bash
subscription-manager repos --enable codeready-builder-for-rhel-9-noarch-rpms
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```
Navigate to the below mentioned directory

```bash
cd /etc/yum.repos.d/
```

Install wget (Package installer)

```bash
yum install wget
```


```bash
wget https://copr.fedorainfracloud.org/coprs/czanik/syslog-ng336/repo/epel-8/czanik-syslog-ng41-epel-8.repo
```
```bash
yum install syslog-ng --nobest
```

Enable and start Syslog

```bash
systemctl enable syslog-ng
systemctl start syslog-ng
```
Check the status of the syslog using the below command

```bash
systemctl status syslog-ng
```
Exit
```bash
exit
```
### Configure Syslog-ng server to receive data from client

Login as root user using below command

```bash
sudo su
```
 

Enable syslog 

```bash
systemctl enable syslog-ng
```


Go to the directory mentioned below

```bash
cd /etc/syslog-ng
```

List the files to check if syslog-ng.conf is available

```bash
ls
```

Edit the syslog-ng.conf file

```bash
vi syslog-ng.conf
```
Do the above steps for both Syslog server and client

Add the below listed lines to receive the data from client. This is configuration for **Syslog-ng server**, give the ip address of client
```bash
source s_network {
  syslog(ip(0.0.0.0) port(514) transport("tcp"));
};
destination d_logs {
  file("/var/log/syslog-ng/logs.txt");
};
log {
  source(s_network);
  destination(d_logs);
};
```
Restart syslog-ng service

```bash
systemctl restart syslog-ng
```

Add the below listed lines to forward the data to server. This is configuration for **Syslog-ng Client**, give the ip address of Syslog Server
```bash
source s_local {
  system();
  internal();
};
destination d_syslog_server {
  syslog("10.1.2.3" transport("tcp"));
};
log {
  source(s_local);
  destination(d_syslog_server);
};
```
Troubleshooting steps

Check for logs in both server and client by check here
```bash
vi /var/log/messages
```

Try to update and install telnet to check if you can reach to server/client
   
   11  sudo yum update
   
   12  sudo yum install telnet -y
   
   13  sudo firewall-cmd --permanent --add-port=514/tcp
   
   14  sudo yum list installed | grep firewalld
   
   15  sudo yum install firewalld -y
   
   16  sudo systemctl status firewalld
   
   17  sudo systemctl start firewalld
   
   18  sudo systemctl enable firewalld
   
   19  sudo firewall-cmd --list-ports
   
   21  sudo firewall-cmd --permanent --add-port=514/tcp
   
   22  sudo firewall-cmd --permanent --add-port=514/udp
   
   23  sudo firewall-cmd --list-ports

   24   telnet 172.31.25.30 514
   
   24  systemctl status syslog-ng

