---
title: Installation for cluster mode
page_title: Installation for cluster mode
font_title: 'fa fa-sitemap'
description: This document will guide you through the cluster mode installation process. 
---

# Installation (distributed/cluster mode)

This document will guide you through the installation process. The ARGO components covered include the following items:

- Consumer service
- Connectors
- Compute Engine
- Web API service
- Web UI service

For a production scale environment we propose three VMs and a Hadoop cluster (distributed version). The proposed setup in this case is the following:

<strong>Node 1 (Consumer service, Connectors and Compute Engine/Hadoop client)</strong>

 - CentOS 6.x
 - 1 CPU
 - 2GB RAM
 - 100 GB Disk

<strong>Node 2 (Datastore and Web API services)</strong>

 - CentOS 6.x
 - 4 CPUs
 - 8GB RAM
 - 60 GB Disk

<strong>Node 3 (Web UI service)</strong>

 - CentOS 7.x
 - 4 CPUs
 - 4GB RAM
 - 40 GB Disk

For this configuration of resources one is stronly encouraged to use the Ansible based deployment roles available on [github][argo-ansible]. The documentation steps provided therein are sufficient in case you choose to use the provided Ansible roles for deployment.
 
If you are not familiar with Ansible or would rather follow the step-by-step guide continue reading. 

[argo-ansible]: https://github.com/ARGOeu/argo-ansible



# Prerequisites

- You will need RHEL 6.x or similar OS (base installation) to proceed. Note that the following instructions have been tested against CentOS 6.x OSes. For the Web UI service (Node 3) an RHEL 7.x or similar OS (base installation) is required. 
- Make sure that on all hosts an ntp client service is configured properly. 
- Configure the OS firewall on Node 2 (Datastore and Web API services) to accept incoming `tcp` connections to ports `443` (https) and `27017` (mongod).
- Configure the OS firewall on Node 3 (Web UI service) to accept incoming `tcp` connections to ports `80` (http) and `443` (https).
- For all nodes you will need an x509 key/certificate pair in order to proceed. 

## Software Repositories

### Trustanchors repository

On all nodes (Node 1, Node 2 and Node 3) create a file named `/etc/yum.repos.d/EGI-trustanchors.repo` and place within it the following contents:

    [EGI-trustanchors]
    name=EGI-trustanchors
    baseurl=http://repository.egi.eu/sw/production/cas/1/current/
    enabled=1
    gpgcheck=1
    gpgkey=http://repository.egi.eu/sw/production/cas/1/GPG-KEY-EUGridPMA-RPM-3

### EPEL repository

On Node 1 and Node 2 install (as root user) the `epel-release` package via yum:

    # yum install epel-release

This package will configure on the host the necessary repository files under `/etc/yum.repos.d`. 

### ARGO repository

On both Node 1 and Node 2 create a new file with filename `/etc/yum.repos.d/argo.repo` and place within it the following contents:

    [argo-prod]
    name=ARGO Product Repository
    baseurl=http://rpm.hellasgrid.gr/mash/centos6-arstats/$basearch
    enabled=0
    gpgcheck=0
    
    [argo-devel]
    name=ARstats Development Repository
    baseurl=http://rpm.hellasgrid.gr/mash/centos6-arstats-devel/$basearch
    enabled=0
    gpgcheck=0

### Cloudera v5 repository

On Node 1 (Sync components, Consumer service and Compute Engine/Hadoop client) you will also need to install the cloudera repository for Hadoop components (clients) to be installed. Create a new file named `/etc/yum.repos.d/cloudera-cdh5.repo` place within it the following contents:

    [cloudera-cdh5]
    # Packages for Cloudera's Distribution for Hadoop, Version 5, on RedHat or CentOS 6 x86_64
    name=Cloudera's Distribution for Hadoop, Version 5
    baseurl=http://archive.cloudera.com/cdh5/redhat/$releasever/$basearch/cdh/5/
    gpgkey = http://archive.cloudera.com/cdh5/redhat/$releasever/$basearch/cdh/RPM-GPG-KEY-cloudera
    gpgcheck = 1
    enabled = 1

### MongoDB v3 repository

On Node 2 (API and Web UI services) you will need to add the MongoDB repository. The name of the file should be `/etc/yum.repos.d/mongodb_3.repo` and its contents should be:

    [mongodb-org-3.0]
    name=MongoDB Repository
    baseurl=http://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.0/x86_64/
    gpgcheck=0
    enabled=1

# Packages

## Node 1 (Consumer service, Connectors and Compute Engine)

Install the `ca-policy-egi-core` meta-rpm with the following command:

    # yum install ca-policy-egi-core

Place the key/certificate pair under the `/etc/grid-security/` folder with the following names and permissions:

* Key file: `/etc/grid-security/hostkey.pem` and permissions: 0400
* Certificate file: `/etc/grid-security/hostcert.pem` and permissions: 0644

### Sync Components and Consumer Service

Next, as root user install the following packages via yum and pip:


    # yum install python-pip
    # pip install --upgrade pymongo
    # yum install avro --enablerepo=argo-prod
    # yum install argo-egi-consumer --enablerepo=argo-prod
    # yum install argo-egi-connectors --enablerepo=argo-prod

The argo-egi-connectors package installs components needed for fetching complimentary to the Compute Engine data from sources of truth (i.e. GOCDB service, POEM service etc). By default the connectors are configured to fetch this information on a daily basis. For configuration details of the connectors visit [this][internal_l1] page. 

[internal_l1]: /guides/egi-connectors/

The argo-egi-consumer service can be configured to connect to one or more message brokers. By default the configuration will connect to `mq.afroditi.hellasgrid.gr` and `mq.cro-ngi.hr`. 

This configuration (having the consumer connected to two or more broker instances) is suggested as it will allow the consumer service to cycle to the next broker in the case the one it is connected to fails for any reason. For further configuration details of the consumer service please refer [here][internal_l2]. 

[internal_l2]: /guides/consumer/

After applying the necessary configurations start the consumer service and add it to appropriate run levels, so that it starts upon the next reboot. 

    # service argo-egi-consumer start
    # chkconfig argo-egi-consumer on

### Compute Engine

As the root user execute the following command to install the compute engine:

    # yum install ar-compute --enablerepo=argo-prod

Edit the `/etc/ar-compute-engine.conf` configuration file and 

- set the value of the `mongo_host` variable to the IP of Node 2. Note that if you have an internal network available among the nodes it should be preffered to use the internal IP. 
- set the value of the `mode` variable to `cluster` like so: `mode=cluster`
- set the values of the `prefilter_clean` and `sync_clean` variables to either `true` of `false`

All configuration options are described in detail [here][internal_l3]

[internal_l3]: /guides/compute/compute-job-configuration/

Under the folder `/etc/cron.d/` place two cronjobs that will handle hourly and daily calculations. 

- for the hourly caclulations edit `/etc/cron.d/ar_job_cycle_hourly` and place the following contents:

    55 * * * * root /usr/libexec/ar-compute/standalone/job_cycle.py -d $(/bin/date --utc  +\%Y-\%m-\%d)

- for the daily caclulations edit `/etc/cron.d/ar_job_cycle_daily` and place the following contents:

    0 0 * * * root /usr/libexec/ar-compute/standalone/job_cycle.py -d $(/bin/date --utc --date '-1 day' +\%Y-\%m-\%d)

### Hadoop client

Configure the Node as a MapReduce and HDFS Hadoop client. 

## Node 2 (Datastore and Web API services)

Place the key/certificate pair under the following folders with the following names and permissions:

* Key file: `/etc/pki/tls/private/localhost.key` and permissions: 0400
* Certificate file: `/etc/pki/tls/certs/localhost.crt` and permissions: 0644

### Datastore

Install MongoDB with the following command:

    # yum install mongodb-org-server-3.0.7 mongodb-org-3.0.7

Edit the `/etc/mongod.conf` file and set the value of the variable `bindIp` either to `0.0.0.0` or to a specific interface (i.e. use the internal NIC in case you have an internal subnet available among the nodes in your setup). 

To start and enable the MongoDB service use the following two commands:

    # service mongod start
    # chkconfig mongod on


### Web API service


Install the ARGO web API service with the following command:

    # yum install argo-web-api --enablerepo=argo-prod

The Web API has a single configuration file: `/etc/argo-web-api.conf`. Make sure that in the `[server]` section the variables `cert` and `key` point to the public x509 certificate and private rsa key files respectively. For further configuration options visit [this][internal_l4] page.

[internal_l4]: /guides/api

Finally, start the service using the following command:

    # start argo-web-api

### Node 3 (Web UI service)

Installation and configuration details for the ARGO Web UI services are available [here][internal_l5]. 

[internal_l5]: /guides/webui/

