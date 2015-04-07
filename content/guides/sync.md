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

Connectors are syncing data on a daily basis. They are aware of the certain entity (EGI, VO), associated jobs and their attributes and are generating and placing files into appropriate job folders. Data is written in a binary avro formated file which is suitable for processing at compute side. Topology, downtimes, weights and POEM profile information all together with a prefiltered status messages, represents an input for argo-compute-engine.

## Installation

Installation narrows down to simply installing the package:
	`yum -y install argo-egi-connectors`

Components require avro package to be installed/available.

Configuration files are placed under `/etc/argo-egi-connectors`, components under `/usr/libexec/argo-egi-connectors`. Cronjobs are placed under `/etc/cron.d` and are configured to be executed once per day. Installation also creates an empty `/var/lib/argo-connectors/EGI` directory where components will put their files.

## Configuration

Configuration of all components is centered around two configuration files: `global.conf` and `customer.conf`. Those files contains some shared config options and sections and are _read by every component_. There's also a third one `poem-connector.conf`, specific only for `poem-connector.py` because it needs some special treatment not available in first two's.

### global.conf

Config file is read by _every_ component because every component needs to fetch host certificate to authenticate to a peer and to find correct avro schema. Config options are case sensitive and whole config file is splitted into a few sections:

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
	TopologyGOCDBGroupOfEndpoints = group_endpoints_%s.avro
	TopologyGOCDBGroupOfGroups = group_groups_%s.avro
	TopologyVOGroupOfEndpoints = group_endpoints_%s.avro
	TopologyVOGroupOfGroups = group_groups_%s.avro
	WeightsGstat = weights_%s.avro

Section lists all the filenames that each component is generating. Directory is purposely omitted because it's implicitly found in next configuration file. Exception is a `PrefilterConsumerFilePath` and `PrefilterPoem` options that tells the `prefilter-egi.py` where to look for its input files. `%s` is a string placeholder that will be replaced by the date timestamp in format `year_month_day`.

### customer.conf

This configuration file lists all EGI' jobs, their attributes and also all VOes and theirs set of jobs and attributes. Job is presented to `argo-compute-engine` as a folder with a set of files that are generated each day and that directs compute engine what status messages to take into account and do calculations upon them. 

#### Directory structure

Job folders are placed under each entity names and correct directory structure and names are _implicitly read from config file_. Segment of configuration file that reflects the creation of directories is for example: 

	[CUSTOMER]
	Jobs = JOB_Test1, JOB_Test2

	[JOB_Test1]
	Dirname = Testing1

	[JOB_Test2]
	Dirname = Testing2

	[VO_VOTesting]
	Dirname = VoName
	VOFeed = http://kosjenka.srce.hr/~eimamagi/ops.feed.xml
	Jobs = JOB_VOTest1, JOB_VOTest2

	[JOB_VOTest1]
	Dirname = Testing1

	[JOB_VOTest2]
	Dirname = Testing2

This will result in the following directory structure:

	/EGI/Testing1
	/EGI/Testing2
	/EGI/VoName/Testing1
	/EGI/VoName/Testing2

So there is EGI as entity, identified with `[CUSTOMER]` section, and one VO VOTesting identified with `[VO_VOTesting]` section and accompanied by `VOFeed`. Multiple VOes can be specified. Each entity have set of jobs listed in `Jobs` option and entity can not exist without associated jobs. The name of the job folder is specified with `Dirname` option of the certain job. JOB\_Test1, identified with `[JOB_Test1]` section, will be named Testing1 and since JOB\_Test1 belongs to EGI, it will be placed under EGI/ directory. JOB\_VOTest2 will be named Testing2, but since it belongs to VOTesting it will be placed under VO's directory specified with `Dirname` option, which is VoName/.

Every connector reads this configuration file because it needs to find out the directory structure where to lay down its files. So `poem-connector.py`, `downtimes-gocdb-connector.py`, `weights-gstat-connector.py`, all of them are writing theirs data in each job folder for EGI and each job folder for each given VO. Topology distinguishes for different entity so exceptions to this are `topology-gocdb-connector.py` and `topology-vo-connector.py`. They are writing data for one entity, EGI or for each given VO, respectively.

#### Job attributes

Besides `Dirname` option that is common for all connectors, some of them have job attributes that are relevant only for them and upon which they are changing their behaviour. For now, that is the case only for `poem-connector.py`, `topology-gocdb-connector.py` and `topology-vo-connector.py`. Furthermore, as topology for each entity is different, so are the attributes and values depending on to which entity job belongs.

##### EGI

	[JOB_Cloudmon]
	Dirname = Cloudmon
	Profiles = CLOUD-MON
	TopoFetchType = ServiceGroups
	TopoSelectGroupOfEndpoints = Monitored:Y, Scope:EGI, Production:Y
	TopoSelectGroupOfGroups = Monitored:Y, Scope:EGI

This is an example of the job for EGI entity. `Profiles` is an attribute relevant to `poem-connector.py` so for this job `poem-connector.py` will write CLOUD-MON profile in Cloudmon job folder under /EGI directory. `Topo*` attributes are relevant for `topology-gocdb-connector.py`.

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

##### VO

	[JOB_BioMedCritical]
	Dirname = Critical
	Profiles = ROC_CRITICAL
	TopoSelectGroupOfGroups = Type:(OPS_Tier, OPS_Site)

This is an example of VO's job. Again, `Profiles` attribute is relevant to `poem-connector.py` which will write ROC_CRITICAL profile in Critical job folder under VO's directory.

Topology is also separated in two abstracts, group of groups and group of service endpoints, but there are no tags needed since VO itself handles what sites and service endpoints to take into account and defines the VO groups they belong to. `TopoSelectGroupOfGroups` is relevant for `topology-vo-connector.py` which will write VO groups that match the selected types into `TopologyVOGroupOfGroups` file.

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

	[DIR]
	OutputDir = /var/lib/argo-connectors/EGI/

	[CUSTOMER]
	Jobs = JOB_Critical, JOB_Cloudmon

	[JOB_Critical]
	Dirname = Critical
	Profiles = ROC_CRITICAL
	TopoFetchType = Sites
	#TopoSelectGroupOfEndpoints = Production:Y, Monitored:Y, Scope:EGI
	TopoSelectGroupOfGroups = Certification:Uncertified, Infrastructure:Test, Scope:EGI

	[JOB_Cloudmon]
	Dirname = Cloudmon
	Profiles = CLOUD-MON
	TopoFetchType = ServiceGroups
	TopoSelectGroupOfEndpoints = Monitored:Y, Scope:EGI, Production:N
	#TopoSelectGroupOfGroups = Monitored:Y, Scope:EGI

	[VO_BioMed]
	Dirname = BioMed
	VOFeed = http://kosjenka.srce.hr/~eimamagi/ops.feed.xml
	Jobs = JOB_BioMedCloudmon, JOB_BioMedCritical

	[JOB_BioMedCritical]
	Dirname = Critical
	Profiles = ROC_CRITICAL
	#TopoSelectGroupOfGroups = Type:(OPS_Tier, OPS_Site)

	[JOB_BioMedCloudmon]
	Dirname = Cloudmon
	Profiles = CLOUD-MON
	#TopoSelectGroupOfGroups = Type:OPS_Tier

VO jobs:

	/var/lib/argo-connectors/EGI/BioMed/Cloudmon/group_endpoints_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed/Cloudmon/group_groups_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed/Cloudmon/poem_sync_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed/Cloudmon/weights_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed/Critical/group_endpoints_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed/Critical/group_groups_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed/Critical/poem_sync_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed/Critical/weights_2015_04_07.avro

EGI jobs:

	/var/lib/argo-connectors/EGI/Cloudmon/group_endpoints_2015_04_07.avro
	/var/lib/argo-connectors/EGI/Cloudmon/group_groups_2015_04_07.avro
	/var/lib/argo-connectors/EGI/Cloudmon/poem_sync_2015_04_07.avro
	/var/lib/argo-connectors/EGI/Cloudmon/weights_2015_04_07.avro
	/var/lib/argo-connectors/EGI/Critical/group_endpoints_2015_04_07.avro
	/var/lib/argo-connectors/EGI/Critical/group_groups_2015_04_07.avro
	/var/lib/argo-connectors/EGI/Critical/poem_sync_2015_04_07.avro
	/var/lib/argo-connectors/EGI/Critical/weights_2015_04_07.avro

Prefilter data:

	/var/lib/argo-connectors/EGI/poem_sync_2015_04_07.out


For EGI's JOB_Critical, we are selecting only those Sites that match `Certification:Uncertified`,  `Infrastructure:Test` and `Scope:EGI`, so in `TopologyGOCDBGroupOfGroups` file there will be only those sites listed:

	 % avro cat /var/lib/argo-connectors/EGI/Critical/group_groups_2015_04_07.avro | tail -n 1
	 {"group": "Russia", "tags": {"scope": "EGI", "infrastructure": "Test", "certification": "Uncertified"}, "type": "NGI", "subgroup": "SU-Protvino-IHEP"}

 For EGI's JOB_Cloudmon, we are selecting only those service endpoints that match `Monitored:Y`, `Scope:EGI`, `Production:N`:

	 % avro cat /var/lib/argo-connectors/EGI/Cloudmon/group_endpoints_2015_04_07.avro
	 {"group": "ROC_RU_SERVICE", "hostname": "ce.ngc6475.ihep.su", "type": "SERVICEGROUPS", "service": "Top-BDII", "tags": {"scope": "EGI", "production": 0, "monitored": 1}}

VO's JOB_BioMedCloudmon requires only CLOUD-MON POEM profile so in `Poem` file you have:

	 % avro cat  /var/lib/argo-connectors/EGI/Cloudmon/poem_sync_2015_04_07.avro | tail -n 5
	 {"profile": "ch.cern.sam.CLOUD-MON", "metric": "eu.egi.cloud.Perun-Check", "service": "egi.Perun", "tags": {"fqan": "", "vo": "ops"}}
	 {"profile": "ch.cern.sam.CLOUD-MON", "metric": "eu.egi.cloud.APEL-Pub", "service": "eu.egi.cloud.accounting", "tags": {"fqan": "", "vo": "ops"}}
	 {"profile": "ch.cern.sam.CLOUD-MON", "metric": "org.nagios.Broker-TCP", "service": "eu.egi.cloud.broker.compss", "tags": {"fqan": "", "vo": "ops"}}
	 {"profile": "ch.cern.sam.CLOUD-MON", "metric": "org.nagios.Broker-TCP", "service": "eu.egi.cloud.broker.proprietary.slipstream", "tags": {"fqan": "", "vo": "ops"}}
	 {"profile": "ch.cern.sam.CLOUD-MON", "metric": "org.nagios.Broker-TCP", "service": "eu.egi.cloud.broker.vmdirac", "tags": {"fqan": "", "vo": "ops"}}

Downtimes:

	% /usr/libexec/argo-egi-connectors/downtimes-gocdb-connector.py -d 2015-04-07
	% find /var/lib/argo-connectors -name '*downtimes*'
	/var/lib/argo-connectors/EGI/Cloudmon/downtimes_2015_04_07.avro
	/var/lib/argo-connectors/EGI/Critical/downtimes_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed/Cloudmon/downtimes_2015_04_07.avro
	/var/lib/argo-connectors/EGI/BioMed/Critical/downtimes_2015_04_07.avro
