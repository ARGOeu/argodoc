---
title: Compute Engine documentation | ARGO
---

# Compute Engine
Argo-compute-engine is the argo component responsible for performing various transformations and computations on the collected metric data in order to provide availability and reliability metrics. The results produced by the compute-engine are forwarded and stored to the configured datastore (MongoDB)

Argo-compute-engine uses the hadoop software stack for performing calculations on the metric data as map-reduce jobs. These jobs can also be run locally (single node mode) in the absence of a proper hadoop cluster. Under the hood the engine uses Apache Pig for map-reduce job submission and execution.

##### Compute engines's main input: metric data

The main engine's input is the metric data collected from the argo-consumer component. These files (according to the default configuration of ar-consumer component) reside to the `/var/lib/ar-consumer/` folder.

Metric data come in the form of avro files and contain timestamped status information about the hostname,service and specific checks (metrics) that are being monitored. A typical item of information int the metric data avro file contains the following mandatory fields:

- `hostname`: the fqdn address of the host being monitored
- `service`: the name of the specific service being monitored
- `metric`: the name of the specific metric (check) of the service that is being monitored
- `timestamp`: time of the monitoring check
-  `status`: status of the metric during the monitoring check

it also includes the following optional fields:

- `monitoring_host`: the fqdn of the monitoring agent 
- `summary`: text containing a summary of the monitoring check
- `message`: text containing the detailed system output message of the monitoring check probe  
- `tags`: array containing optional user defined tags

The current raw avro schema file for the metric data is the following:
```
{"namespace": "argo.avro",
 "type": "record",
 "name": "metric_data",
 "fields": [
    {"name": "timestamp", "type": "string"},
    {"name": "service", "type": "string"},
    {"name": "hostname", "type": "string"},
    {"name": "metric", "type": "string"},
    {"name": "status", "type": "string"},
        {"name": "monitoring_host", "type": ["null", "string"]},
        {"name": "summary", "type": ["null", "string"]},
        {"name": "message", "type": ["null", "string"]},
        {"name": "tags", "type": ["null", {"name" : "Tags",
                                           "type" : "record",
                                           "fields" : [
                                                 {"name" : "roc", "type" : ["null", "string"]},
                                                 {"name" : "vo", "type" : ["null", "string"]},
                                                 {"name" : "vo_fqan", "type" : ["null", "string"]}]}]}
 ]
}
```

##### Compute engine's additional input: topology, profiles, factors 

This core metric data set is processed and transformed with aditional information provided by the argo-connector components. Additional information includes topology, grouping of services, weight factors, lists relevant metrics to be considered, etc. This information is provided per-tenant/per-job in the following path
`/var/lib/ar-sync/{tenant-name}/{job-name}`

for e.g. for tenant-name=T1 and job-name=JobA the correct path with the sync files is as follows
`/var/lib/ar-sync/T1/JobA`

Some sync files that concern the whole enviroment such as the downtime information are provided once in the root ar-sync folder `/var/lib/ar-sync/`

##### Topology files
Topology information is provided by two files: groups of enpoints, groups of groups.

A service endpoint is considered by the engine the simplest item of topology.
Service endpoint combines the information of hostname+service_name. Service endpoints can be grouped together frorming upper level entities named endpoint groups. For example an oranization's geographical IT site that is being monitored can be considered a group of service endpoints. Information for available endpoint groups is contained in the file group_endpoints

###### group_endpoints.avro

The file uses avro format and contains the following fields:

- `group`: The name of the group (e.g. MY-SITE-A)
- `type`: The type of the grouping (e.g. sites)
- `hostname`: The hostname fqdn part info of the specific endpoint contained in the group
- `service`: The service name part info of the specific endpoint contained in the group
- `tags`: (optional) user defined tags providing description metadata

Below is the full description of the group_endpoints.avro specification
```
{"namespace": "argo.avro",
 "type": "record",
 "name": "group_of_service_endpoints",
 "fields": [
        {"name": "type", "type": "string"},
        {"name": "group", "type": "string"},
        {"name": "service", "type": "string"},
        {"name": "hostname", "type": "string"},
        {"name": "tags", "type" : ["null", { "name" : "Tags",
                                             "type" : "record",
                                             "fields" : [
                                                {"name" : "scope", "type" : "string"},
                                                {"name" : "monitored", "type" : "int"},
                                                {"name" : "production", "type" : "int"}]}
                                  ]
        }
 ] 
}
```

###### group_groups.avro

Service endpoint groups can be further grouped in higher-level entities such as for example nation-wide groups of sites etc. The topology information regarding higher-level groups is contained to the group_groups.avro file. 

The file uses avro format and contains the following fields:

- `profile` - name of the profile
- `group`: The name of the group (e.g. MY-SITE-A)
- `type`: The type of the grouping (e.g. sites)
- `hostname`: The hostname fqdn part info of the specific endpoint contained in the group
- `service`: The service name part info of the specific endpoint contained in the group
- `tags`: (optional) user defined tags providing description metadata

Below is the full description of the avro specification:

- `group`: The name of the group (e.g. MY-NATIONAL-GROUP)
- `type`: The type of grouping (e.g. national entities)
- `subgroup`: The name of the lower level group contained (e.g. MY-SITE-A)
- `tags`: (optional) user defined tags providing description metadata

The structure of the specific file gives the ability to define recursively group entities that can be contained as subgroups on other group entities 

an abstract example using cities,nations,continents
```
group: 'Athens', type: 'cities', subgroup:'location-1'
group: 'Athens', type: 'cities', subgroup:'location-2'
group: 'Athens', type: 'cities', subgroup:'location-3'

group: 'Thessaloniki', type: 'cities', subgroup:'location-5'
group: 'Thessaloniki', type: 'cities', subgroup:'location-6'
group: 'Thessaloniki', type: 'cities', subgroup:'location-7'

group: 'Greece', type: 'countries', subgroup: 'Athens'
group: 'Greece', type: 'countries', subgroup: 'Thessaloniki'

group 'Europe', type: 'continents' subgroup: 'Greece'
group 'Europe', type: 'continents' subgroup: 'Croatia'
group 'Europe', type: 'continents' subgroup: 'France'
...etc
```

Below is the full avro specification of the group_groups.avro file:
```
{"namespace": "argo.avro",
 "type": "record",
 "name": "group_groups",
 "fields": [
    {"name": "type", "type": "string"},
    {"name": "group", "type": "string"},
    {"name": "subgroup", "type": "string"},
    {"name": "tags", "type": {"name" : "Tags",
                               "type" : "record",
                               "fields" : [
                                  {"name" : "scope", "type" : "string"},
                                  {"name" : "infrastructure", "type" : "string"},
                                  {"name" : "certification", "type" : "string"}
                                ]}}
 ]
}
```

###### Metric Profiles

Every service type contains a number of metrics that are being checked from the monitoring mechanism. Each metric equals to a specific monitoring check that takes place periodically on the host and has to do with a specific facet of the service's operation (processes,memory,load,files,settings,network etc...)

When wanting to look on the whole status information of the service for a given time it is possible to take into account any number of the metrics available (for e.g. the most critical ones) and compose a view of the service based on those specific metrics selected. This view is dictated by a profile, actually a metric profile which contains information about the service and which metrics are relevant. The metric profile is provided as an avro type file containing the following fields:

- `profile` - name of the profile
- `service` - name of the specific service
- `metric` - name of the metric to be taken into account
- `tags` - (optional) user defined tags 

and the full avro specification:

```
{"namespace": "argo.avro",
 "type": "record",
 "name": "metric_profiles",
 "fields": [
    {"name": "profile", "type": "string"},
    {"name": "service", "type": "string"},
    {"name": "metric", "type": "string"},
    {"name": "tags", "type" : ["null", {"name" : "Tags",
                                        "type" : "record",
                                        "fields" : [
                                          {"name" : "vo", "type" : "string"},
                                          {"name" : "fqan", "type" : "string"}]}
                              ]
    }
 ]
}
```

###### Weights (factors)

Some group items have an assosiated weight information (factors) on how they contribute when are being aggregated on higher level groups. For example hepspec weights for specific sites when they are aggregated on their contribution on national level groups. The weight information is provided in an avro file format containing the following fields:

- `type` : type of the weight (for e.g. hepspec)
- `site` : name of the specific site
- `weight`: number value of the weight

The full avro specification of the weight file:

```
{"namespace": "argo.avro",
 "type": "record",
 "name": "weight_sites",
 "fields": [
    {"name": "type", "type": "string"},
    {"name": "site", "type": "string"},
    {"name": "weight", "type": "string"}
 ]

```

###### Downtimes

Downtime information: the period (start_time --> end_time) in which a specific service endpoint was in scheduled downtime. This information resided in the corresponding downtime avro file. The file has the following fields:

- `hostname` - the hostname fqdn info part of the specific service endpoint
- `service` - the service name info part of the specific service endpoint
- `start_time` - wc3 date/time when the period begins 
- `end_time` - wc3 date/time when the period ends

The full avro specification of the file is provided below
```
{"namespace": "argo.avro",
 "type": "record",
 "name": "downtimes",
 "fields": [
    {"name": "hostname", "type": "string"},
    {"name": "service", "type": "string"},
    {"name": "start_time", "type": "string"},
    {"name": "end_time", "type": "string"}
 ]
}
```

### Tenants and Job definitions
Compute engine can be tenant and job aware. Each tenant must be configured properly both in the argo-connector configuration files and then in the argo-compute-engine configuration. 

For each tenant there is at least one or more computational jobs declared. These jobs allow to perform different calculations on the same metric data in order to produce different a/r results based on topology and metric profiles.

For example regarding a specific tenant we might have two jobs defined based on two different metric profiles: __Job_Critical__, __Job_All__ 

The Job_Critical configuration for example will compute a/r results by taking into account the most critical metrics for each service. The __Job_All__ configuration for example will be more strict by using a profile that takes into account all metrics for each service. 

Each Job configuration includes it's own supplementary data: _topology_, _metric profile_, _availability profile_, _weights_, _downtimes_ etc.

Each tenant has it's own folder which contains job subfolders. Each job subfolder contains daily supplementary data files from argo-connectors. The directory structure resembles the following:

- path_to_argo_connector_data
  + tenantA
    - Job_Critical
    - Job_All

The available tenants and job folder hierarchy is produced by the argo-connector configuration. The compute-engine then is properly configured to recognize the available tenants and jobs and pick the relevant files for each computation. 

### Hadoop client configuration

In order for the engine to be able to connect and submit jobs successfully in a hadoop cluster, proper hadoop client configuration files must be present the installed node (`/etc/hadoop/conf/`)

### Configuration files of Argo-compute-engine

With the installation of Argo-compute-engine component a main configuration file is deployed in `/etc/Argo-compute-engine.conf`. In addition, a directory with supplementary secondary configuration files is created in `/etc/ar-compute/`

##### /etc/Argo-compute-engine.conf (Main configuration file)

This file includes various global parameters used by the engine which are organized in sections, as described next:



#####parameters:
######section: `[default]`

- `mongo_host={IP ADDRESS}`  
specify the ip address of the datastore node (running mongodb)
- `mongo_port={PORT_NUMBER}`   
specify the port number of the datastore node (running mongodb)
- `mode={cluster|local}`   
two available options= **cluster, local**. If specified as **cluster** the engine runs connecting to an available hadoop cluster. If specified as **local**, the engine runs calculating using processes in the local node.
- `serialization={avro|none}`   
two available options= **avro, none**. If specified as **avro**, the engine expects metric and sync data in avro format. If specified as **none**, the engine expects to find metric and sync data in simple text file delimited format
- `prefilter_clean={true|false}`   
If set to **true** local prefilter file will be automatically cleaned after hdfs upload
- `sync_clean={true|false}` 
- If set to **true** uploaded sync files will be automatically cleaned after a job completion

######section: `[jobs]`
In this section we declare the specific tenant used in the installation and the set of jobs available (as we described them above in the [_"Tenants and jobs definitions"_](#tenants-and-job-definitions)). 

- `tenant={TENANT_NAME}`  
specify the name of the tenant used in this installation
- `job_set={JobName1},{JobName2},...{JobNameN}`   
a list already establihed jobs (initially specified in ar-connector components). Names are case-sensitive. For the same tenant multiple jobs can be present. Each job is defined by a set of different argo-connector files (different topologies,metric profiles,weights,etc...). Each job gives the opportunity to calculate a different view base
######section: `[sampling]`
- `s_period={INTEGER}`  
specify the sampling period time in minutes
- `s_interval={INTEGER}`   
specify the sampling interval time in minutes  
***Note:*** the number of samples used in a/r calculations is determined by the s_period/s_interval value. Default values used: **s_period = 1440** (mins) , **s_interval=5** (mins). so number of sample = 1440/5=288 samples.

######section: `[datastore-mapping]`
This section contains various optional parameters used for correctly mapping results to expected datastore fieldnames
- `e_map={fieldname1},{fieldname2}...,{fieldnameN}`  
list for mapping group endpoint availability results to datastore fieldname conventions
- `s_map={fieldname1},{fieldname2}...,{fieldnameN}`  
list for mapping group service availability results to datastore fieldname conventions  
- `n_eg={STRING}`
endpoint group name type used in status detailed calculations
- `n_gg={STRING}`
group of groups name type used in status detailed calculations
- `n_gg={STRING}`
group of groups name type used in status detailed calculations
- `n_alt={STRING}`
mapping of alternative grouping parameter used in status detail calc.
- `n_altf={STRING}`
mapping of alternative grouping parameter used in status detail calc.
- `service_dest={db_name}.{collection_name}`  
destination for storing service a/r reports
- `service_dest={db_name}.{collection_name}`  
destination for storing endpoint grouped a/r reports
- `sdetail_dest={db_name}.{collection_name}`  
destination for storing endpoint grouped a/r reports

#### Contents of /etc/ar-compute directory
As mentioned above secondary configuration files used by the compute-engine are stored to the /etc/ar-compute directory. Here are files describing the set of status state types used in the monitoring engine, algorithmic operations of how to combine those states, availability profiles & configuration files for available jobs.

##### Ops Files (per tenant)
The ops files are json filetypes that are used to describe the available status types encountered in the monitoring enviroment of a tenant and also the availabe algorithmic operations available to use in status aggregations. The filename template used is very specific and is the following:
`{TENANT_NAME}_ops.json`  
for eg. if the tenant name is T1 the corresponding ops filename will be  
`T1_ops.json`

###### Contents of an ops file
An ops file contains:
- the list of available status types 
- which status type is considered as default in missing circumstances
- which status type is considered as default in downtime circumstances
- which status type is considered as default in unknown circumstances
- a list of available operations between statuses expressed in the form of truth tables
 
The available status states in the metric enviroment are expressed in the "states" list as described below 
```
"states":
  [
    "OK",
    "WARNING",
    "UNKNOWN",
    "MISSING",
    "CRITICAL",
    "DOWNTIME"
  ]
```

Based on the availabe states declared in the "states" list are then declared default missing,unknown and downtime states for eg:

```
 "default_down": "DOWNTIME",
 "default_missing": "MISSING",
 "default_unknown": "UNKNOWN",
```

___Note: The importance of the default states___
_Since compute engine gives the ability to define completely custom states based on your monitoring infrastructure output we must also tag some custom states with specific meaning. These states might not be present in the monitoring messages but are produced during computations by the compute engine according to a specific logic. So we need to "tie" some of the custom status we declare to a specific default state of service._

- `"default_down": "DOWNTIME"`, _means that whenever compute engine needs to produce a status for a scheduled downtime will mark it using the "DOWNTIME" state._
- `"default_missing": "MISSING"`, _means whenever compute engine decides that a service status must declared missing (because there is no information provided from the metric data) will mark it using the "MISSING" state._
- `"default_unknown": "UNKNOWN"`, _means whenever compute engine decides that must produce a service status to be considered unknown (for e.g. during recomputation requests) will mark it using the "UNKOWN" state._

The available operations are declared in the operations list using truth tables as follows:

```
"operations":
  {
    "AND":[]
    "OR":[]
  }
```
Each operation consists of a json array used to describe a truth table. For example expanding the above AND operation we see that it consists of a list of dictionary elements

```
"operations":
  {
    "AND":
    [
      { "A":"OK",       "B":"OK",       "X":"OK"       },
      { "A":"OK",       "B":"WARNING",  "X":"WARNING"  },
      { "A":"OK",       "B":"UNKNOWN",  "X":"UNKNOWN"  },
      ...
      ...
      ...
     ]
```
Each element of the json array describes a row of the truth table for eg:  
` { "A":"OK", "B":"WARNING", "X":"WARNING"}`
declares that in an algorithmic AND operation between two status states of *OK* and *WARNING* the result is *WARNING*

In the ops file the user is able to declare any number of availabe monitoring states and any number of available custom operations on those states. The compute-engine picks up this information as a sync file for a job and creates in memory the corresponding truth tables.

##### Config File (per tenant/ per job)
A job config file is a json file that contains specific information needed during the job run such as grouping parameters, the name of the availability profile used and many more.

The file name template of each job config file is the following:  
`{TENANT_NAME}_{JOB_ID}_cfg.json`

If the tenant's name is T1 and the jobs name is JobA then the filename of the config file must be:
`T1_JobA_cfg.json`

###### Contents of an job config file
The configuration file of the job contains mandatory and optional fields with rich information describing the paramaters of the specific job. Some important fields are:
```
"tenant":"tenant_name"`
"job":"job_name",
"aprofile":"availability_profile_name",
"egroup":"endpoint_group_type_name",
"ggroup":"group_of_group_type_name",
"weight":"weight_factor_type_name",
```
In the above snippet are declared the name of the tenant, the name of the job, the name of the specific availability profile used in the job. Also the type of endpoint grouping that will be used is declared here also and the type of upper hierarchical grouping. Also if available here is declared the type of weight factor used for upper level a/r aggregations

In the configuration file are specified the specific tag values that will be used during the job in order to filter metric data.
For eg. 
```
"egroup_tags":{
    "scope":"scope_type",
    "production":"Y",
    "monitored":"Y"
    }
```
In the egroup_tag list are declared values for available tag fields that will be encountered in the endpoint group topology sync file (produced by ar-sync components) 

##### Availability Profile (per tenant / per job)
The availability profile is a json file used per specific job that describes the operations that must take place during aggregating statuses up to endpoint group level. The filename template is specific:
`{TENANT_NAME}_{JOB_NAME}_ap.json`

For eg. if the tenant name = T1 and the job name = JobA the corresponding availability profile name must be:  
`T1_JobA_ap.json`

The information in the availability profile json file is automatically picked up by the compute-engine during computations. 

###### Contents of an availability profile json file

- `"name": "string"`  
the name of the availability profile  
- `"namespace": "string"`  
the name of the namespace used by the profile  
- `"metric_profile": "string"`  
the name of the metric profile linked to this availability profile  
- `"metric_ops": "string"`  
the default operation to be used when aggregating low level metric statuses
- `"group_type": "string"`  
the default endpoint group type used in aggregation

In the availability profile json file also are declared custom grouping of services to be used in the aggregation. The grouping of services are expressed in the json "groups" list see example below:

```
"groups": {
    "my_group_of_services_1": {
      "services":{
        "service_type_A":"OR",
        "service_type_B":"OR"
        },
       "operation":"OR"
    },
    "my_group_of_services_2": {
      "services":{
        "service_type_C":"OR",
        "service_type_D":"OR"
        },
       "operation":"OR"
    },
"operation":"AND"
```

In the above example the service types are grouped in two groups: ***my_group_of_services_1*** and ***my_group_of_services_2***. Each group contains a "service" list containing service types included in the group as fields and the operation values in orded to choose who to aggregate the various instances of a specific service. For example if for ***"service_type_A"*** are 3 service endpoints available they are goint to be aggregated using the OR operation. The ***"operation"*** field under each group of services is used to declare the operation that will be used to aggregate the service types under that group.   
The outer ***"operation"*** field in the root of the json document is used to declare the operation used to aggregate the various groups in order to produce the final endpoint aggregation result.



### Executable Scripts of the standalone edition
In the folder `/usr/libexec/ar-compute/standalone/` reside executable scripts that can be used for uploading metric data and sync data to the hadoop cluster (HDFS Filesystem). 
##### upload_metric.py 
The specific script is used in order to upload daily metric data (relative to a tenant) to HDFS.   
full path: `/usr/libexec/ar-compute/standalone/upload_metric.py`

parameters:
- `-d --date {YYYY-MM-DD}`  
specifies the date of the metric data we want to upload
- `-t --tenant {STRING}`  
a case-sensitive string specifing the name of the tenant


##### upload_sync.py
The specific script is used in order to upload daily sync data (relative to a tenant and a job) to HDFS.
full path: `/usr/libexec/ar-compute/standalone/upload_sync.py` 

parameters:
- `-d --date {YYYY-MM-DD}`  
specifies the date of the sync data we want to upload  
- `-t --tenant {STRING}`  
a case-sensitive string specifing the name of the tenant  
- `-j --job {STRING}`  
a case-sensitive string specifing the name of the job

##### mongo_clean_ar.py
The specific script is used if necessary to clean a/r data from the datastore regarding a specific day
full path: `/usr/libexec/ar-compute/standalone/mongo_clean_ar.py` 

parameters:  
- `-d --date {YYYY-MM-DD}`  
specifies the date (day) to clear the data

optional:
- `-p --profile {STRING}`  
specificy the name of an availability profile. If specified only data a/r data regarding the specified profile will be cleared

##### mongo_clean_status.py
The specific script is used if necessary to clean status detail data from the datastore regarding a specific day
full path: `/usr/libexec/ar-compute/standalone/mongo_clean_status.py` 

parameters:  
- `-d --date {YYYY-MM-DD}`  
specifies the date (day) to clear the data


##### job_ar.py
The specific script is used to submit a specific a/r calculation job for a specific tenant
full path: `/usr/libexec/ar-compute/standalone/job_ar.py` 

parameters:
- `-d --date {YYYY-MM-DD}`  
specifies the date of the sync data we want to upload  
- `-t --tenant {STRING}`  
a case-sensitive string specifing the name of the tenant  
- `-j --job {STRING}`  
a case-sensitive string specifing the name of the job

***Note:*** *the script will take care of automatically calling*  ***upload_sync.py*** *and* ***mongo_clean_ar.py*** *with the correct corresponding parameters in order to ensure the sync data is uploaded before the job and the old datastore entries are cleaned.*

##### job_status_detail.py
The specific script is used to submit a specific status detail job for a specific tenant
full path: `/usr/libexec/ar-compute/standalone/job_status_detail.py` 

parameters:
- `-d --date {YYYY-MM-DD}`  
specifies the date of the sync data we want to upload  
- `-t --tenant {STRING}`  
a case-sensitive string specifing the name of the tenant  

***Note:*** *the script will take care of automatically calling*  ***upload_sync.py*** *and* ***mongo_clean_statys.py*** *with the correct corresponding parameters in order to ensure the sync data is uploaded before the job and the old datastore entries are cleaned.*

##### job_cycle.py
The specific script is used of executing the whole daily cycle of jobs for a specific tenant.
- uploads the available metric data
- calculates the status details
- for each available job calculates a/r results

full path: `/usr/libexec/ar-compute/standalone/job_cycle.py` 

parameters:
- `-d --date {YYYY-MM-DD}`  
specifies the date of the sync data we want to upload  
- `-t --tenant {STRING}`  
a case-sensitive string specifing the name of the tenant  

***Note:*** *the script will take care of automatically calling*  ***upload_metric.py*** *,* ***job_status_detail.py*** *and* ***job_ar.py*** *with the correct corresponding parameters in the correct call-order.* 

##### sync_backup.py
The specific script is used of backing up monthly sync data per tenant (for all available jobs) and store them in the HDFS

full path: `/usr/libexec/ar-compute/standalone/sync_backup.py` 

parameters:
- `-d --date {YYYY-MM-DD}`  
specifies the date of the sync data we want to upload  
- `-t --tenant {STRING}`  
a case-sensitive string specifing the name of the tenant  
