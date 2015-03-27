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

it also includes the following optional fields
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

This core metric data set is processed and transformed with aditional information provided by the argo-sync components. Additional information includes topology, grouping of services, weight factors, lists relevant metrics to be considered, etc. This information is provided per-tenant/per-job in the following path
`/var/lib/ar-sync/{tenant-name}/{job-name}`

for e.g. for tenant-name=T1 and job-name=JobA the correct path with the sync files is as follows
`/var/lib/ar-sync/T1/JobA`

Some sync files that concern the whole enviroment such as the downtime information are provided once in the root ar-sync folder `/var/lib/ar-sync/`

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
- `tenant={TENANT_NAME}`  
specify the name of the tenant used in this installation
- `job_set={JobName1},{JobName2},...{JobNameN}`   
a list already establihed jobs (initially specified in ar-sync components). Names are case-sensitive

######section: `[sampling]`
- `s_period={INTEGER}`  
specify the sampling period time in minutes
- `s_interval={INTEGER}`   
specify the sampling interval time in minutes  
***Note:*** *the number of samples used in a/r calculations is determined by the s_period/s_interval value. Default values used: **s_period = 1440** (mins) , **s_interval=5** (mins). so number of sample = 1440/5=288 samples.*

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
- `d --date {YYYY-MM-DD}`  
specifies the date of the sync data we want to upload  
- `t --tenant {STRING}`  
a case-sensitive string specifing the name of the tenant  
- `j --job {STRING}`  
a case-sensitive string specifing the name of the job

##### mongo_clean_ar.py
The specific script is used if necessary to clean a/r data from the datastore regarding a specific day
full path: `/usr/libexec/ar-compute/standalone/mongo_clean_ar.py` 

parameters:  
- `d --date {YYYY-MM-DD}`  
specifies the date (day) to clear the data

optional:
- `p --profile {STRING}`  
specificy the name of an availability profile. If specified only data a/r data regarding the specified profile will be cleared

##### mongo_clean_status.py
The specific script is used if necessary to clean status detail data from the datastore regarding a specific day
full path: `/usr/libexec/ar-compute/standalone/mongo_clean_status.py` 

parameters:  
- `d --date {YYYY-MM-DD}`  
specifies the date (day) to clear the data


##### job_ar.py
The specific script is used to submit a specific a/r calculation job for a specific tenant
full path: `/usr/libexec/ar-compute/standalone/job_ar.py` 

parameters:
- `d --date {YYYY-MM-DD}`  
specifies the date of the sync data we want to upload  
- `t --tenant {STRING}`  
a case-sensitive string specifing the name of the tenant  
- `j --job {STRING}`  
a case-sensitive string specifing the name of the job

***Note:*** the script will take care of automatically calling  **upload_sync.py** and **mongo_clean_ar.py** with the correct corresponding parameters in order to ensure the sync data is uploaded before the job and the old datastore entries are cleaned.

##### job_status_detail.py
The specific script is used to submit a specific status detail job for a specific tenant
full path: `/usr/libexec/ar-compute/standalone/job_status_detail.py` 

parameters:
- `d --date {YYYY-MM-DD}`  
specifies the date of the sync data we want to upload  
- `t --tenant {STRING}`  
a case-sensitive string specifing the name of the tenant  

***Note:*** the script will take care of automatically calling  **upload_sync.py** and **mongo_clean_statys.py**  with the correct corresponding parameters in order to ensure the sync data is uploaded before the job and the old datastore entries are cleaned.

##### job_cycle.py
The specific script is used of executing the whole daily cycle of jobs for a specific tenant.
- uploads the available metric data
- calculates the status details
- for each available job calculates a/r results

full path: `/usr/libexec/ar-compute/standalone/job_cycle.py` 

parameters:
- `d --date {YYYY-MM-DD}`  
specifies the date of the sync data we want to upload  
- `t --tenant {STRING}`  
a case-sensitive string specifing the name of the tenant  

***Note:*** the script will take care of automatically calling  **upload_metric.py**, **job_status_detail.py** and **job_ar.py** with the correct corresponding parameters in the correct call-order. 

##### sync_backup.py
The specific script is used of backing up monthly sync data per tenant (for all available jobs) and store them in the HDFS

full path: `/usr/libexec/ar-compute/standalone/sync_backup.py` 

parameters:
- `d --date {YYYY-MM-DD}`  
specifies the date of the sync data we want to upload  
- `t --tenant {STRING}`  
a case-sensitive string specifing the name of the tenant  
