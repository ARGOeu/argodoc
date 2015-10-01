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


#### List All metrics:

	/status/{report}/{group_type}/{group_name}/services/{service_type}/endpoints/{hostname}/metrics?[start_time]&[end_time]


#### List a specific metric:

	/status/{report}/{endpoint_group_type}/{endpoint_group_name}/services/{service_type}/endpoints/{hostname}/metrics/{metric_name}?[start_time]&[end_time]


#### Path Parameters

| Type | Description | Required | Default value |
|------|-------------|----------|---------------|
|`report`| name of the report used | YES | |
|`group_type`| type of endpoint group| YES |  |
|`group_name`| name of endpoint group| YES |  |
|`service_type`| type of endpoint group| YES |  |
|`hostname`| hostname of service endpoint| YES |  |
|`metric_name`| name of the metric| NO |  |


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

##### List All metrics: - Example 

###### Request:

URL:

	/status/EGI_CRITICAL/SITES/HG-03-AUTH/services/CREAM-CE/endpoints/cream01.afroditi.gr/metrics?start_time=2015-05-01T00:00:00Z&end_time=2015-05-01T23:59:59Z

Headers:

	x-api-key:"INSERTTENANTKEYHERE"
	Accept:"application/xml"

###### Example Response:

Code:
```
Status: 200 OK
```

###### Reponse body:

	<root>
		<group name="HG-03-AUTH" type="SITES">
			<group name="CREAM-CE" type="service">
				<endpoint name="cream01.afroditi.gr">
					<metric name="emi.cream.CREAMCE-JobSubmit">
						<status timestamp="2015-04-30T23:59:00Z" status="OK"></status>
						<status timestamp="2015-05-01T01:00:00Z" status="CRITICAL"></status>
						<status timestamp="2015-05-01T02:00:00Z" status="OK"></status>
					</metric>
					<metric name="emi.wn.WN-Bi">
						<status timestamp="2015-04-30T22:59:00Z" status="OK"></status>
						<status timestamp="2015-05-01T02:00:00Z" status="OK"></status>
						<status timestamp="2015-05-01T03:00:00Z" status="OK"></status>
					</metric>
				</endpoint>
			</group>
		</group>
	</root>


#### List specific metric 
(`metric_name=emi.cream.CREAM-CE-JobSubmit`):

##### Example Request:

URL:

	/status/EGI_CRITICAL/SITES/HG-03-AUTH/services/CREAM-CE/endpoints/cream01.afroditi.gr/metrics/emi.cream.CREAMCE-JobSubmit?start_time=2015-05-01T00:00:00Z&end_time=2015-05-01T23:59:59Z

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
					<metric name="emi.cream.CREAMCE-JobSubmit">
						<status timestamp="2015-04-30T23:59:00Z" status="OK"></status>
						<status timestamp="2015-05-01T01:00:00Z" status="CRITICAL"></status>
						<status timestamp="2015-05-01T02:00:00Z" status="OK"></status>
					</metric>
				</endpoint>
			</group>
		</group>
	</root>


<a id="2"></a>

## GET: List Service Endpoint Status Timelines

This method may be used to retrieve a specific service endpoint status timeline (applies on a specific service type).

### Input

##### List All endpoints:

	/status/{report}/{group_type}/{group_name}/services/{service_type}/endpoints?[start_time]&[end_time]

##### List a specific endpoint:

	/status/{report}/{endpoint_group_type}/{endpoint_group_name}/services/{service_type}/endpoints/{hostname}?[start_time]&[end_time]

#### Path Parameters

| Type | Description | Required | Default value |
|------|-------------|----------|---------------|
|`report`| name of the report used | YES | |
|`group_type`| type of endpoint group| YES |  |
|`group_name`| name of endpoint group| YES |  |
|`service_type`| type of endpoint group| YES |  |
|`hostname`| hostname of service endpoint| NO |  |

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




<a id="3"></a>

## GET: List Service Status Timelines

This method may be used to retrieve a specific service flavor status timeline (applies for a specific service endpoint group).

### Input

##### List All service types:

	/status/{report}/{group_type}/{group_name}/services[start_time]&[end_time]

##### List a specific service type:

	/status/{report}/{group_type}/{group_name}/services/{service_type}[start_time]&[end_time]

#### Path Parameters

| Type | Description | Required | Default value |
|------|-------------|----------|---------------|
|`report`| name of the report used | YES | |
|`group_type`| type of endpoint group| YES |  |
|`group_name`| name of endpoint group| YES |  |
|`service_type`| type of endpoint group| NO |  |

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

    /status/metrics/msg/{hostname}/{service_flavor}/{metric}?[exec_time]&[job]


#### Parameters

| Type | Description | Required | Default value |
|------|-------------|----------|---------------|
|`exec_time`| UTC time in W3C format| YES |  |
|`job`| Job (view) name | YES |  |

#### Request headers

    x-api-key: "tenant_key_value"

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

