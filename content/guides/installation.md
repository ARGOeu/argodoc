---
title: Installation | ARGO
---

# Installation

This document will guide you through the installation process. The ARGO A/R components include the following items:

- Sync Components
- Consumer service
- Compute Engine
- API service
- Web UI service

These can be co-located on a single machine or more. Note that with respect to the Compute Engine Component there are two options, either to use the standalone version or the distributed version. The latter one requires interconnection with a Hadoop cluster. 

The minimum requirements to meet in case of a single machine (standalone version) are the following:

- 4 CPUs
- 8 GB RAM
- 200 GB Disk

For a production scale environment we propose two machines and a Hadoop cluster (distributed version). This is the setup that is used in production for EGI purposes and needs. In this case the minimum requirements are

- Node 1 (Sync components, Consumer service and Hadoop client)
 - 2 CPUs
 - 4GB RAM
 - 100 GB Disk
- Node 2 (API and Web UI services)
 - 2 CPUs
 - 4GB RAM
 - 100 GB Disk

Instructions on setting up follow:

## Standalone mode

You will need a RHEL 6.x or similar (base installation) to proceed. As a first step make sure that on your host an ntp client service is configured properly. 

### Software Repositories

On your host the next step is to install (as root user) the ar-release package via yum:

    # yum install http://rpm.hellasgrid.gr/mash/centos6-arstats/x86_64/ar-release-1.0.0-3.21.el6.noarch.rpm

This package will configure on the host(s) the repository files under `/etc/yum.repos.d`.

Also install the EPEL repository. This can be done by installing the epel-release package for the appropriate OS. For example:

    # yum install http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm 

You will also need to install the cloudera repository for Hadoop components to be retrieved (although you will not install any Hadoop cluster, some libraries from the Hadoop ecosystem are needed). Under a file named `/etc/yum.repos.d/cloudera-cdh4.repo` place the following contents:

    [cloudera-cdh4]
    name=Cloudera's Distribution for Hadoop, Version 4
    baseurl=http://archive.cloudera.com/cdh4/redhat/6/$basearch/cdh/4/
    gpgkey = http://archive.cloudera.com/cdh4/redhat/6/$basearch/cdh/RPM-GPG-KEY-cloudera
    gpgcheck = 1
    enabled = 1

You will also need to install the EGI trust anchors repository (this is needed on the host for communicating with GOCDB PI service). Under a file named `/etc/yum.repos.d/EGI-trustanchors.repo` place the following contents:

    [EGI-trustanchors]
    name=EGI-trustanchors
    baseurl=http://repository.egi.eu/sw/production/cas/1/current/
    enabled=1
    gpgcheck=1
    gpgkey=http://repository.egi.eu/sw/production/cas/1/GPG-KEY-EUGridPMA-RPM-3

You will finally need to add a MongoDB repository. The name of the file should be `/etc/yum.repos.d/mongodb.repo`) and its contents should be:

    [mongodb]
    name=MongoDB Repository
    baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64/
    gpgcheck=0
    enabled=1

### Sync Components and Consumer Service

First of all install the ca_bundle meta-rpm with the following command:

    # yum install ca-policy-egi-core

As a next step you should request and obtain an x509 host certificate from your national IGTF (Grid-related) approved CA and place the key/certificate pair under `/etc/grid-security/` folder with the following names and permissions:

* Key file: `/etc/grid-security/hostkey.pem` and permissions: 0400
* Certificate file: `/etc/grid-security/hostcert.pem` and permissions: 0644

Next, as root user install the following packages via yum:

    # yum install ar-consumer
    # yum install ar-sync

The ar-sync package installs components needed for refreshing the current status of the monitored system.

The ar-consumer service can be configured to connect to a pool of EGI message brokers. By default the config is set to `mq.afroditi.hellasgrid.gr` and `mq.cro-ngi.hr`. 

This configuration is suggested as it will allow the consumer service to cycle to the next broker in the case the one it is connected to fails for some reason. 

Start consumer service and make sure to add it to appropriate run levels:

    # chkconfig ar-consumer on
    # service ar-consumer start
