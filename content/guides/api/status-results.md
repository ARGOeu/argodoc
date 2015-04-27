---
title: API documentation | ARGO
page_title: API - Status results
font_title: 'fa fa-cogs'
description: API calls for retrieving monitoring status results
---

## API Calls

| Name  | Description | Shortcut |
| GET: List Service Metric Status Timelines | This method may be used to retrieve a specific service metric status timeline (applies on a specific host endpoint and a specific service flavor).|<a href="#1"> Description</a>|
| GET: List Service Endpoint Status Timelines | This method may be used to retrieve a specific service endpoint status timeline (applies on a specific service flavor). | <a href="#2"> Description</a>|
| GET: List Service Flavor Status Timelines |This method may be used to retrieve a specific service flavor status timeline (applies for a specific site). | <a href="#3"> Description</a>|
| GET: List Site Status timelines| This method may be used to retrieve a whole site status timeline. | <a href="#4"> Description</a>|


<a id="1"></a>

## GET: List Service Metric Status Timelines

This method may be used to retrieve a specific service metric status timeline (applies on a specific host endpoint and a specific service flavor).

### Input

    /status/metrics/timeline/{group}?[group_type]&[start_time]&[end_time]&[name]&[host]&[flavor]&[vo]&[profile]

#### Parameters

| Type | Description | Required | Default value |
|`group_type`| `ngi`, `site` or `host`| YES ||
|`start_time`| UTC time in W3C format| YES ||
|`end_time`| UTC time in W3C format| YES| |
|`name`| Metric name| NO| |
|`host`| service host fqdn | NO| |
|`flavor`| service flavor name | NO| |
|`vo`| vo name | NO| |
|`profile`| POEM profile name | NO| |

### Response

Headers: `Status: 200 OK`

#### Response body

    <root>
      <profile name="POEM_PROFILE">
        <metric name="A_METRIC" flavor="A_FLAVOR" host="A_HOSTNAME" vo="A_VO" roc="A_ROC" monitoring_host="A_MONHOST">
          <timeline start_time="2014-10-23T00:00:00Z" end_time="2014-10-24T00:00:00Z">
            <status timestamp="2014-10-23T00:12:34Z" value="OK" />
            <status timestamp="2014-10-23T01:12:28Z" value="OK" />
            <status timestamp="2014-10-23T02:12:31Z" value="CRITICAL" />
            <status timestamp="2014-10-23T03:12:30Z" value="OK" />
            <status timestamp="2014-10-23T04:12:25Z" value="OK" />
            .
            .
            .
            <status timestamp="2014-10-23T23:17:45Z" value="OK" />
          </timeline>
        </metric>
      </profile>
    </root>

<a id="2"></a>

## GET: List Service Endpoint Status Timelines

This method may be used to retrieve a specific service endpoint status timeline (applies on a specific service flavor).

### Input

    /status/timeline/endpoint?[start_time]&[end_time]&[host]&[flavor]&[vo]&[profile]

#### Parameters

| Type | Description | Required | Default value |
|`start_time`| UTC time in W3C format| YES ||
|`end_time`| UTC time in W3C format| YES| |
|`host`| service host fqdn | YES| |
|`flavor`| service flavor name | YES| |
|`vo`| vo name | YES| |
|`profile`| POEM profile name | YES| |

### Response

Headers: `Status: 200 OK`

#### Response body

    <root>
      <profile name="A_POEM">
        <endpoint host="A_HOST" flavor="A_FLAVOR" vo="A_VO" roc="A_ROC" monitoring_host="A_MONHOST">
          <timeline start_time="2014-10-23T00:00:00Z" end_time="2014-10-24T00:00:00Z">
            <status timestamp="2014-10-23T00:12:34Z" value="OK" />
            <status timestamp="2014-10-23T01:12:20Z" value="WARNING" />
            <status timestamp="2014-10-23T02:12:31Z" value="CRITICAL" />
            <status timestamp="2014-10-23T04:12:25Z" value="OK" />
            .
            .
            .
            <status timestamp="2014-10-23T23:17:45Z" value="OK" />
          </timeline>
        </endpoint>
      </profile>
    </root>


<a id="3"></a>

## GET: List Service Flavor Status Timelines

This method may be used to retrieve a specific service flavor status timeline (applies for a specific site).

### Input

    /status/timeline/flavor?[start_time]&[end_time]&[site]&[flavor]&[vo]&[profile]

#### Parameters

| Type | Description | Required | Default value |
|`start_time`| UTC time in W3C format| YES ||
|`end_time`| UTC time in W3C format| YES| |
|`site`| site name | YES| |
|`flavor`| service flavor name | YES| |
|`vo`| vo name | YES| |
|`profile`| POEM profile name | YES| |

### Response

Headers: `Status: 200 OK`

#### Response body

    <root>
      <profile name="A_POEM">
        <flavor name="A_FLAVOR" site="A_SITE-NAME" vo="A_VO" roc="A_ROC" monitoring_host="A_MONHOST">
          <timeline start_time="2014-10-23T00:00:00Z" end_time="2014-10-24T00:00:00Z">
            <status timestamp="2014-10-23T00:12:34Z" value="OK" />
            <status timestamp="2014-10-23T01:12:20Z" value="WARNING" />
            <status timestamp="2014-10-23T02:12:31Z" value="CRITICAL" />
            <status timestamp="2014-10-23T04:12:25Z" value="OK" />
            .
            .
            .
            <status timestamp="2014-10-23T23:17:45Z" value="OK" />
          </timeline>
        </flavor>
      </profile>
    </root>


<a id="4"></a>

## GET: List Site Status timelines

This method may be used to retrieve a whole site status timeline.

### Input

    /status/timeline/site?[start_time]&[end_time]&[site]&[vo]&[profile]


#### Parameters

| Type | Description | Required | Default value |
|`start_time`| UTC time in W3C format| YES ||
|`end_time`| UTC time in W3C format| YES| |
|`site`| site name | YES| |
|`vo`| vo name | YES| |
|`profile`| POEM profile name | YES| |


### Response

Headers: `Status: 200 OK`

#### Response body

    <root>
      <profile name="A_POEM">
        <site name="A_SITE-NAME" vo="A_VO" roc="A_ROC" monitoring_host="A_MONHOST">
          <timeline start_time="2014-10-23T00:00:00Z" end_time="2014-10-24T00:00:00Z">
            <status timestamp="2014-10-23T00:12:34Z" value="OK" />
            <status timestamp="2014-10-23T01:12:20Z" value="WARNING" />
            <status timestamp="2014-10-23T02:12:31Z" value="CRITICAL" />
            <status timestamp="2014-10-23T04:12:25Z" value="OK" />
            .
            .
            .
            <status timestamp="2014-10-23T23:17:45Z" value="OK" />
          </timeline>
        </site>
      </profile>
    </root>


