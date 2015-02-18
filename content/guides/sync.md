---
title: Sync compoments | ARGO
---

# Sync compoments

This document describe the sync compoments.

##  Poem profiles sync

The poem profiles sync download the current poem profiles and stores them to a file.

There are five (5) config files:

- `/etc/ar-sync/poem.conf`,
- `/etc/ar-sync/poem-profile.conf`,
- `/etc/ar-sync/poem-server.conf`,
- `/etc/ar-sync/poem-sync.conf` and
- `/etc/ar-sync/poem-customer.conf`

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

The `/etc/ar-sync/poem-sync.conf` file contains all the information regarding the configurations themselves:

    poemFile = '/etc/ar-sync/poem.conf'
    poemProfileFile = '/etc/ar-sync/poem-profile.conf'
    poemServerFile = '/etc/ar-sync/poem-server.conf'
    poemRequest = '%s/poem/api/0.2/json/metrics_in_profiles/?vo_name=%s'
    hostKey = '/etc/grid-security/hostkey.pem'
    hostCert = '/etc/grid-security/hostcert.pem'
    
    outputDir = '/var/lib/ar-sync'
    avroOutputDir = '/var/lib/ar-sync'
    avroOutputSchema = '/etc/ar-sync/metric_profiles.avsc'

Finally, `/etc/ar-sync/poem-customer.conf` file defines customers, their jobs and directory structure that will be created:

    [DIR]
    OutputDir = /var/lib/ar-sync

    [CUSTOMER_EGI]
    Jobs = JOB_Critical, JOB_Cloudmon

    [JOB_Critical]
    Profiles = ROC_CRITICAL

    [JOB_Cloudmon]
    Profiles = CLOUD-MON

It's a INI file with key, value pairs and sections. Each customer is represented with `[CUSTOMER_name]` section and set of theirs jobs listed in the `Jobs` key. Values of `Jobs` key is a list of `[JOB_name]` sections that are related to a certain job. Job has its set of keys that are actually attributes of job.

>Currently, this config file is only `poem-sync` specific so only POEM Profiles can be specified but the plan is to extend it with the other attributes that refer to other tools from ar-sync package. That means that this config **will not** be only poem-sync specific and it will be read by all other tools from the package.

The name of directory that will be created in an `OutputDir` value is the name of customer. Customer name is a string defined in the section name of the customer, after an underscore. So `CUSTOMER_EGI` will create `EGI` directory in an `OutputDir` value. Following the same logic of naming directories, there will be subdirectories created for each job. So `JOB_Critical` means that there will be `Critical` directory inside of customer's EGI directory.

## Topology sync

Topology is abstracted by the two levels of hierarchy:

- group of groups
- group of service endpoints

Service endpoints are grouped either by the means of Sites or Service groups. Those are listed and represented as an upper level entity of an endpoints - group of groups. Customer can fetch either Sites and their corresponding endpoints or Service groups and their corresponding endpoints per job, but not both of them. Set of tags for further filtering of group of groups is different for Sites and Service groups so it depends on what kind of group of endpoints is being fetched. In contrary, set of tags for groups of endpoints remains the same no matter what type of fetch customer specified. 

`topology-sync` can take config file as an argument and is run per customer's job.

Example of `topology-sync.conf`:

    [FetchType]
    ServiceGroups = True

    [HostCertificate]
    hostKey = /etc/grid-security/hostkey.pem
    hostCert = /etc/grid-security/hostcert.pem

    [OutputDir]
    outputDir = /var/lib/ar-sync/
    avroOutputDir = /var/lib/ar-sync/EGI/Cloudmon

    [AvroSchemas]
    avroOutputGroupOfGroupsSchema = /etc/ar-sync/group_groups.avsc
    avroOutputGroupOfServicesSchema = /etc/ar-sync/group_services.avsc
    avroOutputGroupOfEndpointsSchema = /etc/ar-sync/group_endpoints.avsc

    [SelectGroupOfGroups]
    Monitored = Y
    Scope = Local

    [SelectGroupOfServiceEndpoints]
    Production = Y
    Monitored = Y
    Scope = Local


So, in a FetchType section customer can either specify:

- `ServiceGroups = True` - to fetch Service groups
- `Sites = True` - to fetch Sites

Beside tags, all other options and sections are self-explainable.

### Tags

Tags represent a fine-grained control of what is being written in output files. It's a convenient way of selecting only certain entities, being it Sites, Service groups or Service endpoints based on appropriate criteria. Tags are optional so if a certain tag for a corresponding entity is omitted, than filtering is not done. In that case, it can be considered that entity is fetched for all its values of an omitted tag.

Group of group tags are different for a different type of fetch. Tags and values for a different entities are:

#### Sites

* `Certification  = {Certified, Uncertified, Closed, Suspended, Candidate}`
* `Infrastructure = {Production, Test}`
* `Scope = {EGI, Local}`

#### ServiceGroups

* `Monitored = {Y, N}`
* `Scope = {EGI, Local}`

Tags for selecting the groups of endpoints are:

#### Service Endpoints

* `Production = {Y, N}`
* `Monitored = {Y, N}`
* `Scope = {EGI, Local}`
