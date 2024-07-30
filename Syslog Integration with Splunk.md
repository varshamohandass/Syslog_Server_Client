# Setting up Syslog-ng Server-Client architecure in Redhat Linux

## Install Syslog-ng on Linux (RHEL-9)
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

## Configure Syslog-ng Client to send logs to Syslog-ng Server

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
Add the below listed lines to forward the data to server. This is configuration for **Syslog-ng Client**, give the ip address of Syslog Server
```bash

destination d_network{
        udp("172.31.25.30" port(514));
};

log{
source(s_sys);
destination(d_network);
};
```


Restart syslog-ng service

```bash
systemctl restart syslog-ng
```
### Repeat the client configuration steps in other 3 Syslog-ng clients

## Configure Syslog-ng server to receive data from client

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

Add the below listed lines to receive the data from client and forward them to Splunk. This is configuration for **Syslog-ng server**, In destination stanza give the ip address of Splunk server
```bash
source s_network{
        udp();
};
destination d_from_net{
file("/var/log/from_net");};

destination splunk_tcp {
network("172.31.37.209" transport("tcp") port(5514));
};

log{
source(s_network);
destination(d_from_net);
};

log {
source(s_network);
destination(splunk_tcp);
};

```


Restart syslog-ng service
```bash
systemctl restart syslog-ng
```

### Troubleshooting steps
**Check for Status**
```bash
systemctl status syslog-ng
```
**Try this command for details**
```
journalctl -xeu syslog-ng.service
```

**Try this command to view errors in syslog-ng.conf**
```bash
sudo syslog-ng -Fdev
```

Check for logs in both server and client by check here
```bash
vi /var/log/messages
```
# Receiving data from Syslog-ng server in Splunk

We can see the data in **main** index
