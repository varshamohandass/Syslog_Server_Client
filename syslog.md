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

   10  exit
   
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
   
   24  systemctl status syslog-ng

