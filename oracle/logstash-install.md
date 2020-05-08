# Overview
This document goes through the workflow of installing and configuring a Logstash forwarder on CentOS 7 running Oracle Express v18c and integrating with New Relic Logs

## Logstash Installation

### Validate that Java 8 or 11 is installed

This is a default on Oracle boxes

`java -version`

### Import the Elastic Repo GPG Signing Key

`sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch`

### Build an Elastic 7.x Repo
`sudo -i`

```
cat > /etc/yum.repos.d/logstash-7.repo << EOF
[logstash-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
```

`exit`

### Install Logstash using YUM package manager
`sudo yum install logstash`

### Start Logstash
`sudo systemctl start logstash`

### Install the New Relic Output Plugin
`sudo /usr/share/logstash/bin/logstash-plugin install logstash-output-newrelic`

### Build the Logstash config file
`sudo cp /etc/logstash/logstash-sample.conf /etc/logstash/newrelic-logs.conf`
`sudo mv /etc/logstash/logstash-sample.conf /etc/logstash/logstash-sample.conf.BACKUP`

`sudo vim /etc/logstash/newrelic-logs.conf`

```shell

```