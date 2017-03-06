---
title: Installation for standalone mode
page_title: Installation for standalone mode
font_title: 'fa fa-sitemap'
description: This document will guide you through the standalone mode installation process. 
---

# Installation (standalone mode)

This document will guide you through the installation process. The ARGO components that will be installed on the node include the following items:

- Consumer service
- Connectors
- Compute Engine
- Web API service

For a production environment we propose having one node with the following minimum specifications:

 - 4 CPUs
 - 8GB RAM
 - 100 GB Disk

You are stronly encouraged to use the Ansible based deployment playbook available on [github][argo-ansible].
 
If you are not familiar with Ansible or would rather follow the step-by-step guide continue reading. 

[argo-ansible]: https://github.com/ARGOeu/argo-ansible


# Prerequisites

- You will need a RHEL 6.x or similar OS (base installation) to proceed. Note that the following instructions have been tested against CentOS 6.x OSes.
- Make sure that on your host an ntp client service is configured properly.
- You will need an x509 key/certificate pair in order to proceed. 

The first step is to install (as root user) the EPEL repository definitions via yum:

    # yum install epel-release

Create a new file with filename `/etc/yum.repos.d/argo.repo` and place within it the following contents:

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

You will also need to install the cloudera repository for Hadoop components to be retrieved (although you will not install any Hadoop cluster, some libraries from the Hadoop ecosystem are needed). Under a new file named `/etc/yum.repos.d/cloudera-cdh5.repo` place the following contents:

    [cloudera-cdh5]
    name=Cloudera's Distribution for Hadoop, Version 5
    baseurl=http://archive.cloudera.com/cdh5/redhat/6/x86_64/cdh/5/
    gpgkey =  http://archive.cloudera.com/cdh5/redhat/6/x86_64/cdh/RPM-GPG-KEY-cloudera
    gpgcheck = 1

You will need to add the MongoDB (version 3) repository. The name of the file should be `/etc/yum.repos.d/mongodb_3.repo` and its contents should be:

    [mongodb-org-3.0]
    name=MongoDB Repository
    baseurl=http://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.0/x86_64/
    gpgcheck=0
    enabled=1

Lastly, you need to also install the EGI trustanchors repository (this is required on the host for communicating with topology providing services). Under a new file named `/etc/yum.repos.d/EGI-trustanchors.repo` place the following contents:

    [EGI-trustanchors]
    name=EGI-trustanchors
    baseurl=http://repository.egi.eu/sw/production/cas/1/current/
    enabled=1
    gpgcheck=1
    gpgkey=http://repository.egi.eu/sw/production/cas/1/GPG-KEY-EUGridPMA-RPM-3


# Packages

Install (via `pip`) the latest version of the pymongo library:

    # yum install python-pip
    # pip install --upgrade pymongo

Install avro and the ARGO components:

    # yum install avro --enablerepo=argo-prod
    # yum install argo-egi-consumer --enablerepo=argo-prod
    # yum install argo-egi-connectors --enablerepo=argo-prod
    # yum install ar-compute --enablerepo=argo-prod
    # yum install mongodb-org-server-3.0.7 mongodb-org-3.0.7
    # yum install argo-web-api --enablerepo=argo-prod

# Configuration

## Connectors configuration

The argo-egi-connectors package installs components needed for fetching complimentary to the Compute Engine data from sources of truth (i.e. GOCDB service, POEM service etc). By default the connectors are configured to fetch this information on a daily basis. For configuration details of the connectors visit [this][internal_l1] page. 

[internal_l1]: /guides/egi-connectors/

## Consumer configuration

The argo-egi-consumer service can be configured to connect to one or more message brokers. By default the configuration will connect to `mq.afroditi.hellasgrid.gr` and `mq.cro-ngi.hr`. 

This configuration (having the consumer connected to two or more broker instances) is suggested as it will allow the consumer service to cycle to the next broker in the case the one it is connected to fails for any reason. For further configuration details of the consumer service please refer [here][internal_l2]. 

[internal_l2]: /guides/consumer/

After applying the necessary configurations start the consumer service and add it to appropriate run levels, so that it starts upon the next reboot. 

    # service argo-egi-consumer start
    # chkconfig argo-egi-consumer on


## CE configuration

Edit the `/etc/ar-compute-engine.conf` configuration file and

- set the value of the `mongo_host` variable to `127.0.0.1`
- set the value of the `mode` variable to `local`
- set the values of the `prefilter_clean` and `sync_clean` variables to either `true` of `false`

All configuration options are described in detail [here][internal_l3]

[internal_l3]: /guides/compute/compute-job-configuration/

Under the folder `/etc/cron.d/` place two cronjobs that will handle hourly and daily calculations. 

Under the folder `/etc/cron.d/` place two cronjobs that will handle hourly and daily calculations.

For the daily caclulations edit `/etc/cron.d/ar_job_cycle_daily` and place the following contents:

    0 0 * * * root /usr/libexec/ar-compute/standalone/job_cycle.py -d $(/bin/date --utc --date '-1 day' +\%Y-\%m-\%d)

Optionally, for having hourly caclulations edit `/etc/cron.d/ar_job_cycle_hourly` and place the following contents:

    55 * * * * root /usr/libexec/ar-compute/standalone/job_cycle.py -d $(/bin/date --utc  +\%Y-\%m-\%d)

###  Log files

The compute engine uses by default the system syslog to log any messages. You may change this behaviour by editing the configucation file `/etc/ar-compute-engine.conf`. You may also wish to change the logging level by setting the `log_level` value to your preference.

## Web API and datastore configurations

Edit the `/etc/mongod.conf` file and set the value of the variable `bindIp` to `27.0.0.1`. 

To start and enable the MongoDB service use the following two commands:

    # service mongod start
    # chkconfig mongod on

The Web API has a single configuration file: `/etc/argo-web-api.conf`. Make sure that in the `[server]` section the variables `cert` and `key` point to the public x509 certificate and private rsa key files respectively. For further configuration options visit [this][internal_l4] page.

[internal_l4]: /guides/api

Finally, start the Web API service using the following command:

    # start argo-web-api
