---
title: Installation for cluster mode | ARGO
page_title: Installation for cluster mode
---

# Installation (distributed/cluster mode)

This document will guide you through the installation process. The ARGO components covered include the following items:

- Sync Components
- Consumer service
- Compute Engine
- API service
- Web UI service

For a production scale environment we propose two VMs and a Hadoop cluster (distributed version). The proposed setup in this case is the following:

- Node 1 (Sync components, Consumer service and Hadoop client)
 - 2 CPUs
 - 4GB RAM
 - 100 GB Disk
- Node 2 (API and Web UI services)
 - 2 CPUs
 - 4GB RAM
 - 100 GB Disk


## Cluster mode

### Prerequisites

- You will need RHEL 6.x or similar OS (base installation) to proceed. Note that the following instructions have been tested against CentOS 6.x OSes. 
- Make sure that on each host an ntp client service is configured properly. 
- Configure the OS firewall on Node 2 (API and Web UI services) to accept incoming `tcp` connections to ports `443` (https) and `27017` (mongod).
- For both nodes you will need an x509 key/certificate pair in order to proceed. 

### Software Repositories

On both nodes install (as root user) the `epel` and `argo` release packages via yum:

    # yum install http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
    # yum install http://rpm.hellasgrid.gr/mash/centos6-arstats/i386/ar-release-1.0.0-3.el6.noarch.rpm

These packages will configure on the host the necessary repository files under `/etc/yum.repos.d`.

On Node 1 (Sync components, Consumer service and Hadoop client) you will also need to install the cloudera repository for Hadoop components (clients) to be installed. Under a file named `/etc/yum.repos.d/cloudera-cdh4.repo` place the following contents:

    [cloudera-cdh4]
    name=Cloudera's Distribution for Hadoop, Version 4
    baseurl=http://archive.cloudera.com/cdh4/redhat/6/$basearch/cdh/4/
    gpgkey = http://archive.cloudera.com/cdh4/redhat/6/$basearch/cdh/RPM-GPG-KEY-cloudera
    gpgcheck = 1
    enabled = 1

On the same node will also need to install the EGI trust-anchors repository (this is required on the host for communicating with topology providing services). Under a file named `/etc/yum.repos.d/EGI-trustanchors.repo` place the following contents:

    [EGI-trustanchors]
    name=EGI-trustanchors
    baseurl=http://repository.egi.eu/sw/production/cas/1/current/
    enabled=1
    gpgcheck=1
    gpgkey=http://repository.egi.eu/sw/production/cas/1/GPG-KEY-EUGridPMA-RPM-3

On Node 2 (API and Web UI services) you will finally need to add the MongoDB repository. The name of the file should be `/etc/yum.repos.d/mongodb.repo` and its contents should be:

    [mongodb]
    name=MongoDB Repository
    baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64/
    gpgcheck=0
    enabled=1

### Node 1

Install the `ca-policy-egi-core` meta-rpm with the following command:

    # yum install ca-policy-egi-core

Place the key/certificate pair under `/etc/grid-security/` folder with the following names and permissions:

* Key file: `/etc/grid-security/hostkey.pem` and permissions: 0400
* Certificate file: `/etc/grid-security/hostcert.pem` and permissions: 0644

#### Sync Components and Consumer Service

Next, as root user install the following packages via yum:

    # yum install ar-consumer
    # yum install ar-sync

The ar-sync package installs components needed for refreshing the current status of the monitored system. For configuration details of the sync components visit [this][l1] page. 

[l1]: /guides/sync/

The ar-consumer service can be configured to connect to a pool of message brokers. By default the config is set to `mq.afroditi.hellasgrid.gr` and `mq.cro-ngi.hr`. 

This configuration is suggested as it will allow the consumer service to cycle to the next broker in the case the one it is connected to fails for some reason. For further configuration of the consumer service please refer [here][l2]. 

[l2]: /guides/consumer/

Start consumer service and make sure to add it to appropriate run levels:

    # service ar-consumer start
    # chkconfig ar-consumer on

#### Compute Engine

Install the component:

    # yum install ar-compute

Edit the `/etc/ar-compute-engine.conf` configuration file and 

- set the value of the `mongo_host` variable to the IP of Node 2.
- set the value of the `mode` variable to `cluster` like so: `mode=cluster`

Under the folder `/etc/cron.d/` place two cronjobs that will handle hourly and daily calculations. 

- for the hourly caclulations edit `/etc/cron.d/ar_job_cycle_hourly` and place the following contents:

    55 * * * * root /usr/libexec/ar-compute/standalone/job_cycle.py -d $(/bin/date --utc  +\%Y-\%m-\%d)

- for the daily caclulations edit `/etc/cron.d/ar_job_cycle_daily` and place the following contents:

    0 0 * * * root /usr/libexec/ar-compute/standalone/job_cycle.py -d $(/bin/date --utc --date '-1 day' +\%Y-\%m-\%d)

#### Hadoop client

Configure the Node as a regular Hadoop client. 

### Node 2

Place the key/certificate pair under the following folders with the following names and permissions:

* Key file: `/etc/pki/tls/private/localhost.key` and permissions: 0400
* Certificate file: `/etc/pki/tls/certs/localhost.crt` and permissions: 0644

#### Datastore

Install MongoDB with the following command:

    # yum install mongodb-org-server mongodb-org

Edit the `/etc/mongod.conf` file and set the value of the variable `bind_ip` to `0.0.0.0` like so: `bind_ip=0.0.0.0`.

To start and enable the MongoDB service use the following two commands:

    # service mongod start
    # chkconfig mongod on


### Web API service


Install the ARGO web API service with the following command:

    # yum install ar-web-api

Finally, start the service using the following command:

    # start ar-web-api

