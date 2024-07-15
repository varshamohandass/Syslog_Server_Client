## Configure data flow: Syslog Server using Universal Forwarder --> Intermediate Forwarder --> Indexer

i.Go to your AWS console Create a 3 new instance (syslog-ng, intermediate forwarder, indexer).
ii.Enable the Security groups in your each servers (5514 & 8000-9999) ports.
iii. Install splunk on a server and use it as indexer and configure it to receive data from 9997 port.
iv. Install splunk on a server and use it as Intermediate Forwarder and configure it to receive data from Universal Forwarder and send data to Indexer.
v. Install syslog-ng (syslog-ng server) and install Universal Forwarder in the same server, then configure inputs.conf (to moniter the log file) and outputs.conf (to send logs to Intermediate forwarder).

## 1. Configure Indexer

i.Login to the indexer with your credentials.

ii. Go to Splunk indexer server  **“Settings” >> “Forwarding & receiving” >> Click on “Configure Receiving” >> “New Receiving Port” >> Add “ 9997 ” port >> “Save”**

iii.Finally check if data are sending or not  **Go to “search bar” >> Enter “index=main”**

## 2. Configure to the intermediate forwarder

i. Login to the forwarder with your credentials.

ii.Go to Intermediate Forwarder server  **“Settings” >> “Forwarding & receiving” >> Click on “Configure Forwarding” >> “New Forwarding Host” >> Add your “ indexer Host  ip : 9997 ” >> “Save”**

## 3. Setting up syslog-ng server in redhat linux

**Install Syslog-ng on Linux (RHEL-9)**

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
```
```bash
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

Splunk installation (follow the documentation for both the intermediate forwarder and the Splunk instance)

https://github.com/URahuman/Splunk-System-admin-training/blob/main/Splunk_enterprise_instalation_linux_9.2.2.md

## Install Universal Forwarder in Syslog server

Splunk Universal Forwarder Installation

```bash
https://github.com/URahuman/Splunk_Universal_Forwarder
```

## 2.Configure the Universal forwarder in Syslog-ng server to send data to an Intermediate Forwarder.

Login as root user using below command
```bash
sudo su
```

Provide permission for the Splunk to access syslog files
```bash
chown -R splunk:splunk /var/log/messages
```

Goto inputs.conf file and configure it to monitor the the log file in syslog
```bash
cd /opt/splunkforwarder/etc/apps/search/local
```

Open inputs.conf
```bash
vi inputs.conf
```

Provide the below mentioned configuration in inputs.conf
```bash
[default]
host = universalforwarder

[monitor:///var/log/messages]
disabled = 0
sourcetype = syslogmain
index = main
```

Configure outputs.conf to send to Intermediate forwarder
```bash
cd /opt/splunkforwarder/etc/system/local/
```

Open outputs.conf
```bash
vi outputs.conf
```

Provide the below mentioned configuration in outputs.conf
```bash
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = <intermidiate forwarder ip>:9997

[tcpout-server:// <intermidiate forwarder ip>:9997]
```

#Troubleshooting step

1. Provide the below command when you want to check what are the files has been monitered
```bash
/opt/splunkforwarder/bin/splunk list monitor
```

2. Provide the below command to check how much percentatge of a file has been read
```bash
/opt/splunkforwarder/bin/splunk list inputsstatus
```
