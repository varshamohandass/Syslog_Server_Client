# Setting up Syslog-ng Server-Client architecure in Redhat Linux

## Configure data flow: Syslog Server with and without Universal Forwarder --> Indexer

i.Go to your AWS console Create a 3 new instance (syslog-ng Server,Syslog-ng Client(4), Splunk indexer).

ii.Method - 1 --> Enable the Security groups in your each servers (TCP - Custom - 8000-9999) ports. Enable UDP - Custom - 514 port in Syslog Server. Enable TCP - Custom - 5514 port in Splunk indexer Server. 

iii.Method - 2 --> Enable the Security groups in your each servers (TCP - Custom - 8000-9999) ports. Enable UDP - Custom - 514 port in Syslog Server.

iv. Install splunk on a server and use it as indexer and configure it to receive data from 9997 port.

v. Install syslog-ng (syslog-ng server) and install Universal Forwarder in the same server, then configure inputs.conf (to moniter the log file) and outputs.conf (to send logs to Indexer).

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

### Method 1 (Receiving Data from Syslog-ng Client and Sending it to the Splunk Server using Syslog-ng.conf file)
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

### Method 2 (Receiving Data from Syslog-ng Client and Sending it to a file, then monitor the file using Universal Forwarder to forward logs to the Splunk Server)

### Configure Indexer

i.Login to the indexer with your credentials.

ii. Go to Splunk indexer server  **“Settings” >> “Forwarding & receiving” >> Click on “Configure Receiving” >> “New Receiving Port” >> Add “ 9997 ” port >> “Save”**

iii.Finally check if data are sending or not  **Go to “search bar” >> Enter “index=<index name>”**

## Configure Syslog-ng server to receive data from client


**Install Universal Forwarder in Syslog server**

Splunk Universal Forwarder Installation

```bash
https://github.com/URahuman/Splunk_Universal_Forwarder
```

** Configure the Universal forwarder in Syslog-ng server to send data to an Intermediate Forwarder**

**Login as root user using below command**
```bash
sudo su
```

**Provide permission for the Splunk to access syslog files**
```bash
chown -R splunk:splunk /var/log/messages
```

**Goto inputs.conf file and configure it to monitor the the log file in syslog**
```bash
cd /opt/splunkforwarder/etc/apps/search/local
```

**Open inputs.conf**
```bash
vi inputs.conf
```

**Provide the below mentioned configuration in inputs.conf**
```bash
[default]
host = universalforwarder

[monitor:///var/log/messages]
disabled = 0
sourcetype = syslogmain
index = <index name>
```

**Configure outputs.conf to send to Splunk Server/Indexer**
```bash
cd /opt/splunkforwarder/etc/system/local/
```

Open outputs.conf
```bash
vi outputs.conf
```

**Provide the below mentioned configuration in outputs.conf**
```bash
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = <indexer ip>:9997

[tcpout-server:// <indexer ip>:9997]
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
