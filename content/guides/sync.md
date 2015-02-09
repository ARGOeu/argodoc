---
title: Sync compoments | ARGO
---

# Sync compoments

This document describe the sync compoments.

##  Poem profiles sync

The poem profiles sync download the current poem profiles and stores them to a file.

There are four (4) config files: 

- `/etc/ar-sync/poem.conf`, 
- `/etc/ar-sync/poem-profile.conf`, 
- `/etc/ar-sync/poem-server.conf` and
- `/etc/ar-sync/poem-sync.conf`

This application requires host certificates and expects the certificate file to be installed at:
`/etc/grid-security/hostkey.pem`
`/etc/grid-security/hostcert.pem`

The `/etc/ar-sync/poem.conf` config should look like this:

    snf-624922.vm.okeanos.grnet.gr;ops

Each line in the `/etc/ar-sync/poem.conf` file defines a poem server from which to load profiles and a list of VOs to define which profiles are used.

In the default configuration profiles are loaded from one poem server: `mon.egi.eu`, and only ops VO's profile are used.

The `/etc/ar-sync/poem-profile.conf` config file looks like this:

    CH.CERN.SAM.ROC
    CH.CERN.SAM.ROC_OPERATORS
    CH.CERN.SAM.ROC_CRITICAL
    CH.CERN.SAM.OPS_MONITOR
    CH.CERN.SAM.OPS_MONITOR_CRITICAL
    CH.CERN.SAM.GLEXEC
    CH.CERN.SAM.CLOUD-MON

Each line in the `/etc/ar-sync/poem-profile.conf` file defines full name of filtered profiles list. So the list of profiles loaded form the servers are filtered according to this list.

The `/etc/ar-sync/poem-server.conf` config file looks like this:

    URL=http://mon.egi.eu/nagios-roles.conf;ch.cern.sam.ROC,ch.cern.sam.ROC_OPERATORS,ch.cern.sam.ROC_CRITICAL,ch.cern.sam.GLEXEC
    ALL:opsmon.egi.eu;ch.cern.sam.OPS_MONITOR,ch.cern.sam.OPS_MONITOR_CRITICAL
    ALL:cloudmon.egi.eu;ch.cern.sam.CLOUD-MON

Finally, the `/etc/ar-sync/poem-sync.conf` file contains all the information regarding the configurations themselves:

    poemFile = '/etc/ar-sync/poem.conf'
    poemProfileFile = '/etc/ar-sync/poem-profile.conf'
    poemServerFile = '/etc/ar-sync/poem-server.conf'
    poemRequest = '%s/poem/api/0.2/json/metrics_in_profiles/?vo_name=%s'
    hostKey = '/etc/grid-security/hostkey.pem'
    hostCert = '/etc/grid-security/hostcert.pem'
    
    outputDir = '/var/lib/ar-sync'
    avroOutputDir = '/var/lib/ar-sync'
    avroOutputSchema = '/etc/ar-sync/metric_profiles.avsc'

