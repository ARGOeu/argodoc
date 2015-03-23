---
title: API documentation | ARGO
page_title: API documentation 
font_title: 'fa fa-cogs'
description: This document describes the API service, using the HTTP application protocol. This API uses XML as the primary exchange format.
---

## API Data

| Base URL | <code>https://\<host\>:\<port\>/api/v1/</code> |
| **Default Port**         | <code>443</code>  |
| **Central instance of the ARGO production service** |  <code>https://snf-629551.vm.okeanos.grnet.gr/api/v1/</code> |

## GET method

### Groups Availability and Reliability

#### Input

    /group_availability?[start_time]&[end_time]&[availability_profile]&[group_type]&[granularity]&[infrastructure]&[production]&[monitored]&[certification]&[format]&[group_name]

- mandatory
  - `start_time`: UTC time in W3C format 
  - `end_time`: UTC time in W3C format
  - `availability_profile`: Name of the high level profile concatenated with the profile namespace. Each availability profile matches one POEM profile.
  - `group_type`: Valid values are: `site`, `ngi` or `vo`
- optional
  - `granularity`: Possible values: `DAILY`, `MONTHLY` (default: `DAILY`)
  - `infrastructure`: Filter results based on the name of the infrastructure the site belongs to (default: `Production`)
  - `production`: Filter results based on whether they are production sites or not. Possible values: `true`, `false` (default: `true`)
  - `monitored`: Filter results based on whether they are monitored sites or not. Possible values: `true`, `false`. (default: `true`)
  - `certification`: Filter results based on the certification status of each site (default: `Certified`)
  - `format`: Default is `XML` (only xml available right now, so the parameter is void thus deactivated for the time being)
  - `group_name`: Site, NGI or VO name. If no name is specified then all sites, NGIs or VOs are retrieved. 

#### Response

Headers: `Status: 200 OK`

##### Response body for `group_type=site`

    <root>
      <Profile name="A_PROFILE_NAME">
        <Site site="Site-Name" NGI="NGI-Name" infastructure="Type" scope="Scope" site_scope="Any" production="Y" monitored="Y" certification_status="Certified">
          <Availability timestamp="YYYY-MM-DD" availability="1" reliability="1"/>
          <Availability timestamp="YYYY-MM-DD" availability="1" reliability="1"/>
        </Site>
        <Site site="Site-Name-Another" NGI="NGI-Name-Another" infastructure="Type" scope="Scope" site_scope="Any" production="Y" monitored="Y" certification_status="Certified">
          <Availability timestamp="YYYY-MM-DD" availability="1" reliability="1"/>
          <Availability timestamp="YYYY-MM-DD" availability="1" reliability="1"/>
        </Site>
        .
        .
        .
      </Profile>
    </root>

##### Response body for `group_type=ngi`

    <root>
      <Profile name="A_PROFILE" namespace="a_namespace.grid.auth.gr">
        <Ngi NGI="A_NGI">
          <Availability timestamp="2013-08-01" availability="87.64699776723847" reliability="87.64699776723847">
          </Availability>
          <Availability timestamp="2013-08-02" availability="87.63642636198455" reliability="87.63642636198455">
          </Availability>
          <Availability timestamp="2013-08-03" availability="11.307937916910474" reliability="11.307937916910474">
          </Availability>
          .
          .
          .
          <Availability timestamp="2013-08-09" availability="87.69028873148349" reliability="92.9126400880786">
          </Availability>
        </Ngi>
      </Profile>
    </root>


##### Response body for `group_type=vo`

     <root>
       <Profile name="A_PROFILE">
         <Vo VO="ops">
           <Availability timestamp="2013-11-09" availability="100" reliability="100"></Availability>
           <Availability timestamp="2013-11-19" availability="50" reliability="100"></Availability>
           <Availability timestamp="2013-12-09" availability="100" reliability="100"></Availability>
         </Vo>
       </Profile>
     </root>

### NGI Availability and Reliability

#### Input

    /group_availability

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


