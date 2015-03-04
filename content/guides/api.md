---
title: API documentation | ARGO
page_title: API documentation 
font_title: 'fa fa-cogs'
description: This document describes the API service, using the HTTP application protocol. This API uses XML as the primary exchange format.
---

## API Data

| Base URL | <code>https://\<host\>:\<port\>/api/v1/</code> |
| **Default Port**         | <code>8080</code>  |
| **Central instance of the ARGO production service** |  <code>https://snf-629551.vm.okeanos.grnet.gr:8080/api/v1/</code> |

## GET method

### Input

    /api/v1/service_availability_in_profile?[start_time]&[end_time]&[vo_name]&[profile_name]&[group_type]&[availability_period]&[output]&[namespace]&[group_name]&[service_flavour]&[service_hostname] 

- mandatory
  - `start_time`: UTC time in W3C format 
  - `end_time`: UTC time in W3C format
  - `vo_name`: Name of the VO requested. May appear more than once. (eg: ops)
  - `profile_name`: Name of the profile requested. May appear more than once. (eg: CMS_CRITICAL)
  - `group_type`: Type of the aggregation grouping requested.  May appear more than once. (eg: CMS_Site)
  - `availability_period`: Results granularity. Possible values: 'HOURLY', 'DAILY', 'WEEKLY', 'MONTHLY'
- optional
  - `output`: Type of the output format. Default: XML, Possible values: XML, json
  - `namespace`: Profile namespace. May appear more than once. (eg: ch.cern.sam)
  - `group_name`: Site name. May appear more than once
  - `service_flavour`: Service flavour name. May appear more than once. (eg: SRMv2)
  - `service_hostname`: Service hostname. May appear more than once. (eg: ce202.cern.ch)

### Output 

    <pre>
      <root>
        <Profile name="A_PROFILE_NAME" namespace="a_namespace.grid.auth.gr" defined_by_vo_name="A_VO">
          <Service hostname="a_host.grid.auth.gr" type="A_FLAVOR" flavor="A_FLAVOR">
            <Availability timestamp="2013-08-01T00:00:00Z" availability="1" reliability="1" maintenance="-1"></Availability>
            <Availability timestamp="2013-08-01T01:00:00Z" availability="1" reliability="1" maintenance="-1"></Availability>
            <Availability timestamp="2013-08-01T02:00:00Z" availability="1" reliability="1" maintenance="-1"></Availability>
            <Availability timestamp="2013-08-01T03:00:00Z" availability="1" reliability="1" maintenance="-1"></Availability>
            <Availability timestamp="2013-08-01T04:00:00Z" availability="1" reliability="1" maintenance="-1"></Availability>
            <Availability timestamp="2013-08-01T05:00:00Z" availability="1" reliability="1" maintenance="-1"></Availability>
            <Availability timestamp="2013-08-01T06:00:00Z" availability="1" reliability="1" maintenance="-1"></Availability>
            <Availability timestamp="2013-08-01T07:00:00Z" availability="1" reliability="1" maintenance="-1"></Availability>
            <Availability timestamp="2013-08-01T08:00:00Z" availability="1" reliability="1" maintenance="-1"></Availability>
            .
            .
            .
          </Service>
        </Profile>
      </root>
    </pre>


