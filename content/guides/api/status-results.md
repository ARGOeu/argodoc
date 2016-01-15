---
title: API documentation | ARGO
page_title: API - Status results
font_title: 'fa fa-cogs'
description: API calls for retrieving monitoring status results
---

## API Calls

| Name  | Description | Shortcut |
|-------|-------------|----------|
| GET: List Service Metric Status Timelines | This method may be used to retrieve a specific service metric status timeline (applies on a specific service endpoint).|<a href="#1"> Description</a>|
| GET: List Service Endpoint Status Timelines | This method may be used to retrieve a specific service endpoint status timeline (applies on a specific service type). | <a href="#2"> Description</a>|
| GET: List Service  Status Timelines |This method may be used to retrieve a specific service type status timeline (applies for a specific service endpoint group). | <a href="#3"> Description</a>|
| GET: List Endpoint Group Status Timelines| This method may be used to retrieve endpoint group status timelines. | <a href="#4"> Description</a>|


<a id="1"></a>

## GET: List Service Metric Status Timelines

This method may be used to retrieve a specific service metric status timeline (applies on a specific host endpoint and a specific service flavor).

### Input

    /status/metrics/timeline/{group}?[start_time]&[end_time]&[vo]&[profile]&[group_type]


#### List All metrics:

	/status/{report}/{group_type}/{group_name}/services/{service_type}/endpoints/{hostname}/metrics?[start_time]&[end_time]


#### List a specific metric:

	/status/{report}/{endpoint_group_type}/{endpoint_group_name}/services/{service_type}/endpoints/{hostname}/metrics/{metric_name}?[start_time]&[end_time]


#### Path Parameters

| Type | Description | Required | Default value |
|`start_time`| UTC time in W3C format| YES |  |
|`end_time`| UTC time in W3C format| YES |  |
|`vo`| vo name | NO | `ops` |
|`profile`| POEM profile name | NO| `ch.cern.sam.ROC_CRITICAL` |
|`group_type`| `ngi`, `site` or `host`| NO | `site` |

Depending on the `group_type`, `{group}` is the name of the group (for example `NGI_GRNET` when `group_type=ngi`, `HG-03-AUTH` when `group_type=site` or `cream.afroditi.hellasgrid.gr` when `group_type=host`). 


#### Url Parameters

| Type | Description | Required | Default value |
|------|-------------|----------|---------------|
|`start_time`| UTC time in W3C format| YES |  |
|`end_time`| UTC time in W3C format| YES |  |

___Notes___:
`group_type` and `group_name` in the specific request refer always to endpoint groups (eg. `SITES`).
when `metric_name` is supplied, the request returns results for a specific metric. Else returns results for all available metrics for the specific __endpoint__ (and __report__)

#### Headers

	x-api-key: "tenant_key_value"
	Accept: "application/xml" or "application/json"

### Responses


#### Response Code

```
Status: 200 OK
```

#### Response body

##### `group_type=site`

    <root>
      <profile name="ch.cern.sam.ROC_CRITICAL">
        <group name="ops" type="vo">
          <group name="NGI_GRNET" type="ngi">
            <group name="HG-03-AUTH" type="site">
              <group name="CREAM-CE" type="service_type">
                <host name="cream.afroditi.hellasgrid.gr">
                  <metric name="emi.cream.CREAMCE-JobSubmit">
                    <status timestamp="2015-02-03T22:58:39Z" status="OK"/>
                    <status timestamp="2015-02-04T02:23:45Z" status="OK"/>
                    .
                    .  status results
                    .
                  </metric>
                  .
                  .  metrics
                  .
                </host>
                .
                .  hosts (endpoints)
                .
              </group>
              .
              .  service flavors
              .
            </group>
          </group>
        </group>
      </profile>
    </root>

##### `group_type=host`

    <root>
      <profile name="ch.cern.sam.ROC_CRITICAL">
        <group name="ops" type="vo">
          <group name="NGI_GRNET" type="ngi">
            <group name="HG-03-AUTH" type="site">
              <group name="CREAM-CE" type="service_type">
                <host name="cream.afroditi.hellasgrid.gr">
                  <metric name="emi.cream.CREAMCE-JobSubmit">
                    <status timestamp="2015-02-03T22:58:39Z" status="OK"/>
                    <status timestamp="2015-02-04T02:23:45Z" status="OK"/>
                    .
                    .  status results
                    .
                  </metric>
                  .
                  .  other metrics (i.e. probes) results
                  .
                </host>
              </group>
            </group>
          </group>
        </group>
      </profile>
    </root>



<a id="2"></a>

## GET: List Service Endpoint Status Timelines

This method may be used to retrieve a specific service endpoint status timeline (applies on a specific service type).

### Input

    /status/endpoints/timeline/{group}?[start_time]&[end_time]&[vo]&[profile]&[group_type]


	/status/{report}/{group_type}/{group_name}/services/{service_type}/endpoints?[start_time]&[end_time]

##### List a specific endpoint:

	/status/{report}/{endpoint_group_type}/{endpoint_group_name}/services/{service_type}/endpoints/{hostname}?[start_time]&[end_time]

#### Path Parameters

| Type | Description | Required | Default value |
|`start_time`| UTC time in W3C format| YES | |
|`end_time`| UTC time in W3C format| YES| |
|`vo`| vo name | NO | `ops` |
|`profile`| POEM profile name | NO | `ch.cern.sam.ROC_CRITICAL` |
|`group_type`| `ngi` or `site` | NO | `site` |

Depending on the `group_type`, `{group}` is the name of the group (for example `HG-03-AUTH` when `group_type=site` or `NGI_GRNET` when `group_type=ngi`). 


#### Url Parameters

| Type | Description | Required | Default value |
|------|-------------|----------|---------------|
|`start_time`| UTC time in W3C format| YES |  |
|`end_time`| UTC time in W3C format| YES |  |

___Notes___:
`group_type` and `group_name` in the specific request refer always to endpoint groups (eg. `SITES`).
when `hostname` is supplied, the request returns results for a specific endpoint. Else returns results for all available metrics for the specific __endpoint__ (and __report__)

#### Headers

	x-api-key: "tenant_key_value"
	Accept: "application/xml" or "application/json"

#### Response Code

```
Status: 200 OK
```

### Response body

##### List All metrics:

###### Example Request:

URL:

	/status/EGI_CRITICAL/SITES/HG-03-AUTH/services/CREAM-CE/endpoints?start_time=2015-05-01T00:00:00Z&end_time=2015-05-01T23:59:59Z

Headers:

	x-api-key:"INSERTTENANTKEYHERE"
	Accept:"application/xml"

###### Example Response:

Code:

```
Status: 200 OK
```

Reponse body:

	<root>
		<group name="HG-03-AUTH" type="SITES">
			<group name="CREAM-CE" type="service">
				<endpoint name="cream01.afroditi.gr">
					<status timestamp="2015-04-30T23:59:00Z" status="OK"></status>
					<status timestamp="2015-05-01T01:00:00Z" status="CRITICAL"></status>
					<status timestamp="2015-05-01T02:00:00Z" status="OK"></status>
				</endpoint>
				<endpoint name="cream02.afroditi.gr">
					<status timestamp="2015-04-30T23:59:00Z" status="OK"></status>
					<status timestamp="2015-05-01T01:00:00Z" status="CRITICAL"></status>
					<status timestamp="2015-05-01T02:00:00Z" status="OK"></status>
				</endpoint>
			</group>
		</group>
	</root>



#### List specific endpoint
(`hostname=cream01.afroditi.gr`):

##### Example Request:

URL:

	/status/EGI_CRITICAL/SITES/HG-03-AUTH/services/CREAM-CE/endpoints/cream01.afroditi.gr?start_time=2015-05-01T00:00:00Z&end_time=2015-05-01T23:59:59Z

Headers:

	x-api-key:"INSERTTENANTKEYHERE"
	Accept:"application/xml"

##### Example Response:

Code:
```
Status: 200 OK
```

Reponse body:

	<root>
		<group name="HG-03-AUTH" type="SITES">
			<group name="CREAM-CE" type="service">
				<endpoint name="cream01.afroditi.gr">
					<status timestamp="2015-04-30T23:59:00Z" status="OK"></status>
					<status timestamp="2015-05-01T01:00:00Z" status="CRITICAL"></status>
					<status timestamp="2015-05-01T02:00:00Z" status="OK"></status>
				</endpoint>
			</group>
		</group>
	</root>


##### `group_type=site`

    <root>
      <profile name="ch.cern.sam.ROC_CRITICAL">
        <group name="ops" type="vo">
          <group name="NGI_GRNET" type="ngi">
            <group name="HG-03-AUTH" type="site">
              <group name="CREAM-CE" type="service_type">
                <endpoint hostname="cream.afroditi.hellasgrid.gr" service="CREAM-CE">
                  <status timestamp="2015-02-04T00:00:00Z" status="OK"/>
                  <status timestamp="2015-02-04T01:28:44Z" status="UNKNOWN"/>
                  <status timestamp="2015-02-04T02:23:45Z" status="OK"/>
                  .
                  . status results for given endpoint
                  .
                </endpoint>
              </group>
              <group name="SRMv2" type="service_type">
                <endpoint hostname="se01.afroditi.hellasgrid.gr" service="SRMv2">
                  <status timestamp="2015-02-04T00:00:00Z" status="OK"/>
                  <status timestamp="2015-02-04T21:57:05Z" status="OK"/>
                </endpoint>
              </group>
              <group name="Site-BDII" type="service_type">
                <endpoint hostname="sbdii.afroditi.hellasgrid.gr" service="Site-BDII">
                  <status timestamp="2015-02-04T00:00:00Z" status="OK"/>
                  <status timestamp="2015-02-04T22:13:56Z" status="OK"/>
                </endpoint>
              </group>
            </group>
          </group>
        </group>
      </profile>
    </root>

##### `group_type=ngi`

    
    <root>
      <profile name="ch.cern.sam.ROC_CRITICAL">
        <group name="ops" type="vo">
          <group name="NGI_GRNET" type="ngi">
            <group name="GR-01-AUTH" type="site">
              <group name="CREAM-CE" type="service_type">
                <endpoint hostname="cream01.grid.auth.gr" service="CREAM-CE">
                  <status timestamp="2015-02-04T00:00:00Z" status="OK"/>
                  <status timestamp="2015-02-04T22:28:21Z" status="OK"/>
                  .
                  . status results for given endpoint
                  .
                </endpoint>
              </group>
              <group name="SRMv2" type="service_type">
              .
              .  other service flavors
              .
            </group>
            .
            .  other sites within specified NGI
            .
          </group>
        </group>
      </profile>
    </root>

<a id="3"></a>

## GET: List Service Status Timelines

This method may be used to retrieve a specific service flavor status timeline (applies for a specific service endpoint group).

### Input

    /status/services/timeline/{group}?[start_time]&[end_time]&[vo]&[profile]&[group_type]


	/status/{report}/{group_type}/{group_name}/services[start_time]&[end_time]

##### List a specific service type:

	/status/{report}/{group_type}/{group_name}/services/{service_type}[start_time]&[end_time]

#### Path Parameters

| Type | Description | Required | Default value |
|`start_time`| UTC time in W3C format| YES ||
|`end_time`| UTC time in W3C format| YES| |
|`vo`| vo name | NO | `ops` |
|`profile`| POEM profile name | NO | `ch.cern.sam.ROC_CRITICAL` |
|`group_type`| `site` or `ngi` | NO | `site` |

Depending on the `group_type`, `{group}` is the name of the group (for example `HG-03-AUTH` when `group_type=site` or `NGI_GRNET` when `group_type=ngi`).


#### Url Parameters

| Type | Description | Required | Default value |
|------|-------------|----------|---------------|
|`start_time`| UTC time in W3C format| YES |  |
|`end_time`| UTC time in W3C format| YES |  |

___Notes___:
`group_type` and `group_name` in the specific request refer always to endpoint groups (eg. `SITES`).
when `service_name` is supplied, the request returns results for a specific service type. Else returns results for all available service types for the specific __endpoint_group__ (and __report__)

#### Headers

	x-api-key: "tenant_key_value"
	Accept: "application/xml" or "application/json"

#### Response Code

```
Status: 200 OK
```

### Response body

##### List All service types:

###### Example Request:

URL:

	/status/EGI_CRITICAL/SITES/HG-03-AUTH/services?start_time=2015-05-01T00:00:00Z&end_time=2015-05-01T23:59:59Z

Headers:

	x-api-key:"INSERTTENANTKEYHERE"
	Accept:"application/xml"

###### Example Response:

Code:
```
Status: 200 OK
```

Reponse body:

	<root>
		<group name="HG-03-AUTH" type="SITES">
			<group name="CREAM-CE" type="service">
				<status timestamp="2015-04-30T23:59:00Z" status="OK"></status>
				<status timestamp="2015-05-01T01:00:00Z" status="CRITICAL"></status>
				<status timestamp="2015-05-01T02:00:00Z" status="OK"></status>
			</group>
			<group name="SRMv2" type="service">
				<status timestamp="2015-04-30T23:59:00Z" status="OK"></status>
				<status timestamp="2015-05-01T01:00:00Z" status="CRITICAL"></status>
				<status timestamp="2015-05-01T02:00:00Z" status="OK"></status>
			</group>
		</group>
	</root>


#### List specific service type 
(`service_type=CREAM-CE`):

##### Example Request:

URL:

	/status/EGI_CRITICAL/SITES/HG-03-AUTH/services/CREAM-CE?start_time=2015-05-01T00:00:00Z&end_time=2015-05-01T23:59:59Z

Headers:

	x-api-key:"INSERTTENANTKEYHERE"
	Accept:"application/xml"


##### Example Response:

Code:
```
Status: 200 OK
```

Reponse body:

	<root>
		<group name="HG-03-AUTH" type="SITES">
			<group name="CREAM-CE" type="service">
				<status timestamp="2015-04-30T23:59:00Z" status="OK"></status>
				<status timestamp="2015-05-01T01:00:00Z" status="CRITICAL"></status>
				<status timestamp="2015-05-01T02:00:00Z" status="OK"></status>
			</group>
		</group>
	</root>

<a id="4"></a>

## GET: List Endpoint Group Status Timelines

This method may be used to retrieve status timelines for endpoint groups.

### Input

#### List All endpoint groups of specific type:

	/status/{report}/{group_type}[start_time]&[end_time]

##### List a specific endpoint group of specific type:


	/status/{report}/{group_type}/{group_name}[start_time]&[end_time]

#### Path Parameters

| Type | Description | Required | Default value |
|------|-------------|----------|---------------|
|`report`| name of the report used | YES | |
|`group_type`| type of endpoint group| YES |  |
|`group_name`| name of endpoint group| NO |  |

#### Url Parameters

| Type | Description | Required | Default value |
|------|-------------|----------|---------------|
|`start_time`| UTC time in W3C format| YES |  |
|`end_time`| UTC time in W3C format| YES |  |

___Notes___:
`group_type` and `group_name` in the specific request refer always to endpoint groups (eg. `SITES`).
when `group_name` is supplied, the request returns results for a specific endpoint group. Else returns results for all available endpoint groups of the specific __group_type__

#### Headers

	x-api-key: "tenant_key_value"
	Accept: "application/xml" or "application/json"

#### Response Code

```
Status: 200 OK
```

### Response body

##### List All endpoint groups:

###### Example Request:

URL:

	/status/EGI_CRITICAL/SITES?start_time=2015-05-01T00:00:00Z&end_time=2015-05-01T23:59:59Z

Headers:

	x-api-key:"INSERTTENANTKEYHERE"
	Accept:"application/xml"


###### Example Response:

Code:
```
Status: 200 OK
```

Reponse body:

	<root>
		<group name="HG-03-AUTH" type="SITES">
			<status timestamp="2015-05-01T01:00:00Z" status="CRITICAL"></status>
			<status timestamp="2015-05-01T02:00:00Z" status="OK"></status>
			<status timestamp="2015-05-01T05:00:00Z" status="OK"></status>
		</group>
		<group name="HG-01-AUTH" type="SITES">
			<status timestamp="2015-05-01T01:00:00Z" status="CRITICAL"></status>
			<status timestamp="2015-05-01T02:00:00Z" status="OK"></status>
			<status timestamp="2015-05-01T05:00:00Z" status="OK"></status>
		</group>
	</root>


#### List specific endpoint group 
(`group_name=HG-03-AUTH`):

##### Example Request:

URL:

	/status/EGI_CRITICAL/SITES/HG-03-AUTH?start_time=2015-05-01T00:00:00Z&end_time=2015-05-01T23:59:59Z

Headers:

	x-api-key:"INSERTTENANTKEYHERE"
	Accept:"application/xml"


##### Example Response:

Code:
```
Status: 200 OK
```

Reponse body:

	<root>
		<group name="HG-03-AUTH" type="SITES">
			<status timestamp="2015-05-01T01:00:00Z" status="CRITICAL"></status>
			<status timestamp="2015-05-01T02:00:00Z" status="OK"></status>
			<status timestamp="2015-05-01T05:00:00Z" status="OK"></status>
		</group>
	</root>


## GET: List Metric Results

This method may be used to retrieve a detailed metric result.

### Input

    /status/sites/timeline/{group}?[start_time]&[end_time]&[site]&[vo]&[profile]



#### Parameters

| Type | Description | Required | Default value |
|`start_time`| UTC time in W3C format| YES ||
|`end_time`| UTC time in W3C format| YES| |
|`vo`| vo name | NO | `ops` |
|`profile`| POEM profile name | NO | `ch.cern.sam.ROC_CRITICAL` |
|`group_type`| `site` or `ngi` | NO | `site` |

Depending on the `group_type`, `{group}` is the name of the group (for example `HG-03-AUTH` when `group_type=site` or `NGI_GRNET` when `group_type=ngi`).


### Response

Headers: `Status: 200 OK`

#### Response body

    <root>
       <job name="ROC_CRITICAL">
         <group name="NGI_GRNET" type="ngi">
           <group name="HG-03-AUTH" type="site">
             <group name="CREAM-CE" type="service_type">
               <endpoint name="cream01.afroditi.gr">
                 <metric name="emi.cream.CREAMCE-JobSubmit">
                   <status timestamp="2015-05-01T01:00:00Z" status="">
                     <summary>Cream status is CRITICAL</summary>
                     <message>Cream job submission test failed!</message>
                   </status>
                 </metric>
               </endpoint>
             </group>
           </group>
         </group>
       </job>
     </root>
    <root>

