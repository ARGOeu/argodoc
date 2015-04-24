---
title: EGI connectors | ARGO
page_title: EGI connectors
font_title: 'fa fa-refresh'
description: This document describes the available connectors for data in EGI infrastructure.
---

## Description

argo-egi-connectors is a bundle of connectors/sync components for various data sources established in EGI infrastructure, most notably GOCDB (EGI topology, downtimes), but there's also support for fetching alternative EGI topology via various VO feeds, weights information via GStat service and POEM metric profiles.

Bundle consists of the following connectors: `topology-gocdb-connector.py`, `topology-vo-connector.py`, `downtimes-gocdb-connector.py`, `weights-gstat-connector.py`, `poem-connector.py`

Additionally there is `prefilter-egy.py` component whose role is to filter out the messages coming from the argo-egi-consumer.

Connectors are syncing data on a daily basis. They are aware of the certain customer, associated jobs and their attributes and are generating and placing files into appropriate job folders. Data is written in a binary avro formated file which is suitable for processing at compute side. Topology, downtimes, weights and POEM profile information all together with a prefiltered metric results (status messages), represents an input for argo-compute-engine.

## Installation

Installation narrows down to simply installing the package:
	`yum -y install argo-egi-connectors`

Components require avro package to be installed/available.

Configuration files are placed under `/etc/argo-egi-connectors`, components under `/usr/libexec/argo-egi-connectors`. Cronjobs are placed under `/etc/cron.d` and are configured to be executed once per day. Installation also creates an empty `/var/lib/argo-connectors/EGI` directory where components will put their files.

## Configuration

Configuration of all components is centered around two configuration files: `global.conf` and `customer.conf`. Those files contains some shared config options and sections and are _read by every component_. There's also a third one `poem-connector.conf`, specific only for `poem-connector.py` because it needs some special treatment not available in first two's.

### global.conf

Config file is read by _every_ component because every component needs to fetch host certificate to authenticate to a peer and to find correct avro schema. Config options are case insensitive and whole config file is splitted into a few sections:

	[DEFAULT]
	SchemaDir = /etc/argo-egi-connectors/schemas/

Every component generates output file in an avro binary format. This section points to a directory that holds all avro schemas. 

	[URL]
	WeightsGstat = http://gstat2.grid.sinica.edu.tw/gstat/summary/json/

`weights-gstat-connector.py` fetchs its data from a GStat service.

	[Authentication]
	HostKey = /etc/grid-security/hostkey.pem
	HostCert = /etc/grid-security/hostcert.pem

Each component that talks to GOCDB or POEM peer authenticates itself with a host certificate.

	[AvroSchemas]
	DowntimesGOCDB = %(SchemaDir)s/downtimes.avsc
	Poem = %(SchemaDir)s/metric_profiles.avsc
	Prefilter = %(SchemaDir)s/metric_data.avsc
	TopologyGOCDBGroupOfEndpoints = %(SchemaDir)s/group_endpoints.avsc
	TopologyGOCDBGroupOfGroups = %(SchemaDir)s/group_groups-sites.avsc
	TopologyGOCDBGroupOfServices = %(SchemaDir)s/group_groups-services.avsc
	TopologyVOGroupOfEndpoints = %(SchemaDir)s/vo_group_endpoints.avsc
	TopologyVOGroupOfGroups = %(SchemaDir)s/vo_group_groups.avsc
	WeightsGstat = %(SchemaDir)s/weight_sites.avsc

This section, together with a [DEFAULT] section, constitutes the full path of avro schema file for each component.

	[Output]
	DowntimesGOCDB = downtimes_%s.avro
	Poem = poem_sync_%s.avro
	Prefilter = prefilter_%s.avro
	PrefilterConsumerFilePath = /var/lib/ar-consumer/ar-consumer_log_%s.avro
	PrefilterPoem = poem_sync_%s.out
	PrefilterPoemNameMapping = poem_name_mapping.cfg
	TopologyGroupOfEndpoints = group_endpoints_%s.avro
	TopologyGroupOfGroups = group_groups_%s.avro
	WeightsGstat = weights_%s.avro

Section lists all the filenames that each component is generating. Directory is purposely omitted because it's implicitly found in next configuration file. Exception is a `PrefilterConsumerFilePath` and `PrefilterPoem` options that tells the `prefilter-egi.py` where to look for its input files. `%s` is a string placeholder that will be replaced by the date timestamp in format `year_month_day`.

### customer.conf

This configuration file lists all customers, their jobs and appropriate attributes. Job is presented to `argo-compute-engine` as a folder with a set of files that are generated each day and that directs compute engine what metric results to take into account and do calculations upon them. 

#### Directory structure

Job folders for each customer are placed under the same `EGI/` directory and names are read from config file. Segment of configuration file that reflects the creation of directories is for example: 

	[DIR]
	OutputDir = /var/lib/argo-connectors/EGI/

	[CUSTOMER_EGI]
	Jobs = JOB_Test1, JOB_Test2

	[JOB_Test1]
	Dirname = EGI_Testing1

	[JOB_Test2]
	Dirname = EGI_Testing2


	[CUSTOMER_NGI]
	Jobs = Job_Test3, JOB_Test4

	[JOB_Test3]
	Dirname = NGI_Testing1

	[JOB_Test4]
	Dirname = NGI_Testing2

This will result in the following jobs directories:

	/var/lib/argo-connectors/EGI/EGI_Testing1
	/var/lib/argo-connectors/EGI/EGI_Testing2
	/var/lib/argo-connectors/EGI/NGI_Testing1
	/var/lib/argo-connectors/EGI/NGI_Testing2

So there are two customers, EGI and NGI, each one identified with its `[CUSTOMER_*]` section. `CUSTOMER_` is a section keyword and must be specified when one wants to define a new customer section. Each customer has set of jobs listed in `Jobs` option and customer can not exist without associated jobs. The name of the job folder is specified with `Dirname` option of the certain job so JOB\_Test1, identified with `[JOB_Test1]` section, will be named EGI\_Testing1 and it will be placed under EGI/ directory. 

Every connector reads this configuration file because it needs to find out the job directory name where to lay down its files. So `poem-connector.py`, `downtimes-gocdb-connector.py`, `weights-gstat-connector.py`, all of them are writing theirs data in each job directory for each customer. Topology for EGI is different than one for the VO so exceptions to this are `topology-gocdb-connector.py` and `topology-vo-connector.py`. They are writing data for a job based on the value of the `TopoName` job attribute.

#### Job attributes

Besides `Dirname` option that is common for all connectors, some of them have job attributes that are relevant only for them and upon which they are changing their behaviour. For now, that is the case only for `poem-connector.py`, `topology-gocdb-connector.py` and `topology-vo-connector.py`. Furthermore, as there are two kind of topologies, there are also two set of job attributes and values.

##### GOCDB topology

	[JOB_EGICloudmon]
	Dirname = EGI_Cloudmon
	Profiles = CLOUD-MON
	TopoName = GOCDB
	TopoFetchType = ServiceGroups
	TopoSelectGroupOfEndpoints = Monitored:Y, Scope:EGI, Production:Y
	TopoSelectGroupOfGroups = Monitored:Y, Scope:EGI

This is an example of the job that fetchs topology from GOCD since `TopoName` attribute is set to `GOCDB`. `Profiles` is an attribute relevant to `poem-connector.py` so for this job `poem-connector.py` will write CLOUD-MON profile in EGI_Cloudmon job folder under /EGI directory. `Topo*` attributes are relevant for `topology-gocdb-connector.py`.

Topology is separated in two abstracts:

- group of groups
- group of service endpoints

Service endpoints are grouped either by the means of _Sites_ or _Service groups_. Those are listed and represented as an upper level abstract of group of service endpoints - group of groups. Customer can fetch either _Sites_ and their corresponding endpoints or _Service groups_ and their corresponding endpoints per job, but not both of them. What is being fetched is specified with `TopoFetchType` option/job attribute. For each abstract there will be written either `TopologyGOCDBGroupOfGroups` or `TopologyGOCDBGroupOfServices`  and `TopologyVOGroupOfEndpoints` files into appropriate job folder. `TopoSelectGroupOfGroups` and `TopoSelectGroupOfEndpoints` options are used for further filtering. Values are set of tags used for picking up matching entity existing in the given abstract. Tags for group of groups are different for Sites and Service groups. In contrary, set of tags for groups of endpoints remains the same no matter what type of fetch customer specified. 

So, in a `TopoFetchType` option customer can either specify:

- `ServiceGroups` - to fetch Service groups
- `Sites` - to fetch Sites

###### Tags

Tags represent a fine-grained control of what is being written in output files. It's a convenient way of 
selecting only certain entities, being it Sites, Service groups or Service endpoints based on appropriate
criteria. Tags are optional so if a certain tag for a corresponding entity is omitted, than filtering is 
not done. In that case, it can be considered that entity is fetched for all its values of an omitted tag.

Group of group tags are different for a different type of fetch. Tags and values for a different entities
are:

####### Sites

* Certification = `{Certified, Uncertified, Closed, Suspended, Candidate}`
* Infrastructure = `{Production, Test}`
* Scope = `{EGI, Local}`

####### ServiceGroups

* Monitored = `{Y, N}`
* Scope = `{EGI, Local}`

Tags for selecting group of endpoints are:

####### Service Endpoints

* Production = `{Y, N}`
* Monitored = `{Y, N}`
* Scope = `{EGI, Local}`

##### VO topology

	[DEFAULT]
	BioMed = http://kosjenka.srce.hr/~eimamagi/ops.feed.xml

	[JOB_BioMedCritical]
	Dirname = BioMed_Critical
	Profiles = ROC_CRITICAL
	TopoName = VOFeed
	VOFeed = %(BioMed)s
	TopoSelectGroupOfGroups = Type:(OPS_Tier, OPS_Site)

This is an example of the job that is fetching topology from provided VO feed since `TopoName` attribute is set to `VOFeed`. Again, `Profiles` attribute is relevant to `poem-connector.py` which will write ROC\_CRITICAL profile in BioMed\_Critical job folder. `Topo*` attributes are relevant for `topology-vo-connector.py`.

VO topology is also separated in two abstracts, group of groups and group of service endpoints, but there are no tags needed since VO itself handles what sites and service endpoints to take into account and defines the VO groups they belong to. `TopoSelectGroupOfGroups` is relevant for `topology-vo-connector.py` which will write VO groups that match the selected types into `TopologyGroupOfGroups` file.

### poem-connector.conf

This configuration file is central configuration for `poem-connector.py` whose role is:

- fetch all defined POEM profiles from each POEM server specified
- prepare and layout data needed for `prefilter-egi.py`

#### POEM profiles fetch

Config file is splitted into a few sections:

	[PoemServer]
	Host = snf-624922.vm.okeanos.grnet.gr
	VO = ops

This section defines the URL where POEM server is located and all VOes for which POEM profiles will be fetched. Multiple POEM servers can be specified by defining multiple POEM server sections:

	[PoemServer1]
	Host = poem1
	VO = vo1, vo2

	[PoemServer2]
	Host = poem2
	VO = vo3, vo4

POEM profile can be defined on multiple POEM servers. Each POEM server can further extend it with a custom combinations of metrics and service flavours. To distinguish POEM profile defined on multiple POEM servers, namespace must be used. One must be aware of the namespace that POEM server exposes and specify it in `FetchProfiles` section:

	[FetchProfiles]
	List = CH.CERN.SAM.ROC, CH.CERN.SAM.ROC_OPERATORS, CH.CERN.SAM.ROC_CRITICAL, CH.CERN.SAM.OPS_MONITOR, CH.CERN.SAM.OPS_MONITOR_CRITICAL, CH.CERN.SAM.GLEXEC, CH.CERN.SAM.CLOUD-MON

#### Prefilter data

`poem-connector.py` also generates plaintext `PrefilterPoem` file on a daily basis and places it under EGI/ directory. Content of the file is controlled in `[PrefilterData]` section:

	[PrefilterData]
	AllowedNGI = http://mon.egi.eu/nagios-roles.conf
	AllowedNGIProfiles = ch.cern.sam.ROC, ch.cern.sam.ROC_OPERATORS, ch.cern.sam.ROC_CRITICAL, ch.cern.sam.GLEXEC
	AllNGI1 = opsmon.egi.eu
	AllNGIProfiles1 = ch.cern.sam.OPS_MONITOR, ch.cern.sam.OPS_MONITOR_CRITICAL
	AllNGI2 = cloudmon.egi.eu
	AllNGIProfiles2 = ch.cern.sam.CLOUD-MON

`AllowedNGI` option defines remote config file that states all allowed NGIes and corresponding nagios boxes. All of them will be expanded and listed together with the information from `AllowedNGIProfiles` POEM profiles (metrics, service flavours, VOes). 

`AllNGI1` option is similar in sense that it will extended specified nagios box with the information from `AllNGIProfiles1` POEM profiles.

With all these informations written in `PrefilterPoem` file, `prefilter-egi.py` can do its work, so it will filter consumer messages if:

- message that enter into broker network doesn't come from allowed NGI or nagios box for certain NGI is incorrectly specified 
- status message is response to metric not found in a fetched service flavour 
- status message's service flavour is not registered in any fetched POEM profile
- status message is registered for different VO, not specified in `VO` option of `[PoemServer]` section 

## Examples

customer.conf:

	[DEFAULT]
	BioMed = http://kosjenka.srce.hr/~eimamagi/ops.feed.xml

	[DIR]
	OutputDir = /var/lib/argo-connectors/EGI/

	[CUSTOMER_EGI]
	Jobs = JOB_EGICritical, JOB_EGICloudmon, JOB_BioMedCloudmon, JOB_BioMedCritical

	[JOB_EGICritical]
	Dirname = EGI_Critical
	Profiles = ROC_CRITICAL
	TopoName = GOCDB
	TopoFetchType = Sites
	#TopoSelectGroupOfEndpoints = Production:Y, Monitored:Y, Scope:EGI
	TopoSelectGroupOfGroups = Certification:Uncertified, Infrastructure:Test, Scope:EGI

	[JOB_EGICloudmon]
	Dirname = EGI_Cloudmon
	Profiles = CLOUD-MON
	TopoName = GOCDB
	TopoFetchType = ServiceGroups
	TopoSelectGroupOfEndpoints = Monitored:Y, Scope:EGI, Production:N
	#TopoSelectGroupOfGroups = Monitored:Y, Scope:EGI

	[JOB_BioMedCritical]
	Dirname = BioMed_Critical
	Profiles = ROC_CRITICAL
	TopoName = VOFeed
	VOFeed = %(BioMed)s
	#TopoSelectGroupOfGroups = Type:(OPS_Tier, OPS_Site)

	[JOB_BioMedCloudmon]
	Dirname = BioMed_Cloudmon
	Profiles = CLOUD-MON
	TopoName = VOFeed
	VOFeed = %(BioMed)s
	#TopoSelectGroupOfGroups = Type:OPS_Tier

Customer jobs:

	/var/lib/argo-connectors/EGI/BioMed_Cloudmon/group_endpoints_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed_Cloudmon/group_groups_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed_Cloudmon/poem_sync_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed_Cloudmon/weights_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed_Critical/group_endpoints_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed_Critical/group_groups_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed_Critical/poem_sync_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed_Critical/weights_2015_04_07.avro
	/var/lib/argo-connectors/EGI/EGI_Cloudmon/group_endpoints_2015_04_07.avro
	/var/lib/argo-connectors/EGI/EGI_Cloudmon/group_groups_2015_04_07.avro
	/var/lib/argo-connectors/EGI/EGI_Cloudmon/poem_sync_2015_04_07.avro
	/var/lib/argo-connectors/EGI/EGI_Cloudmon/weights_2015_04_07.avro
	/var/lib/argo-connectors/EGI/EGI_Critical/group_endpoints_2015_04_07.avro
	/var/lib/argo-connectors/EGI/EGI_Critical/group_groups_2015_04_07.avro
	/var/lib/argo-connectors/EGI/EGI_Critical/poem_sync_2015_04_07.avro
	/var/lib/argo-connectors/EGI/EGI_Critical/weights_2015_04_07.avro

Prefilter data:

	/var/lib/argo-connectors/EGI/poem_sync_2015_04_07.out


For customer's job JOB_EGICritical, we are selecting only those sites that match `Certification:Uncertified`,  `Infrastructure:Test` and `Scope:EGI`, so in `TopologyGroupOfGroups` file there will be only those sites listed:

	 % avro cat /var/lib/argo-connectors/EGI/EGI_Critical/group_groups_2015_04_07.avro | tail -n 1
	 {"group": "Russia", "tags": {"scope": "EGI", "infrastructure": "Test", "certification": "Uncertified"}, "type": "NGI", "subgroup": "SU-Protvino-IHEP"}

 For customer's JOB_EGICloudmon, we are selecting only those service endpoints that match `Monitored:Y`, `Scope:EGI`, `Production:N`:

	 % avro cat /var/lib/argo-connectors/EGI/EGI_Cloudmon/group_endpoints_2015_04_07.avro
	 {"group": "ROC_RU_SERVICE", "hostname": "ce.ngc6475.ihep.su", "type": "SERVICEGROUPS", "service": "Top-BDII", "tags": {"scope": "EGI", "production": 0, "monitored": 1}}

JOB_BioMedCloudmon requires only CLOUD-MON POEM profile so in `Poem` file you have:

	 % avro cat  /var/lib/argo-connectors/EGI/BioMed_Cloudmon/poem_sync_2015_04_07.avro | tail -n 5
	 {"profile": "ch.cern.sam.CLOUD-MON", "metric": "eu.egi.cloud.Perun-Check", "service": "egi.Perun", "tags": {"fqan": "", "vo": "ops"}}
	 {"profile": "ch.cern.sam.CLOUD-MON", "metric": "eu.egi.cloud.APEL-Pub", "service": "eu.egi.cloud.accounting", "tags": {"fqan": "", "vo": "ops"}}
	 {"profile": "ch.cern.sam.CLOUD-MON", "metric": "org.nagios.Broker-TCP", "service": "eu.egi.cloud.broker.compss", "tags": {"fqan": "", "vo": "ops"}}
	 {"profile": "ch.cern.sam.CLOUD-MON", "metric": "org.nagios.Broker-TCP", "service": "eu.egi.cloud.broker.proprietary.slipstream", "tags": {"fqan": "", "vo": "ops"}}
	 {"profile": "ch.cern.sam.CLOUD-MON", "metric": "org.nagios.Broker-TCP", "service": "eu.egi.cloud.broker.vmdirac", "tags": {"fqan": "", "vo": "ops"}}

Downtimes:

	% /usr/libexec/argo-egi-connectors/downtimes-gocdb-connector.py -d 2015-04-07
	% find /var/lib/argo-connectors -name '*downtimes*'
	/var/lib/argo-connectors/EGI/EGI_Cloudmon/downtimes_2015_04_07.avro
	/var/lib/argo-connectors/EGI/EGI_Critical/downtimes_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed_Cloudmon/downtimes_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed_Critical/downtimes_2015_04_07.avro
