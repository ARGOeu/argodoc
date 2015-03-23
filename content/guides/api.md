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

## GET methods

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

    /service_flavor_availability?[start_time]&[end_time]&[profile]&[granularity]&[format]&[flavor]&[site]

- mandatory
  - `start_time`: UTC time in W3C format
  - `end_time`: UTC time in W3C format
  - `profile`: Name of the POEM profile according to which the result was calculated.
- optional
  - `granularity`: Possible values: `DAILY`, `MONTHLY`. (default: `DAILY`)
  - `format`: Default is `XML` (only xml available right now, so the parameter is void thus deactivated for the time being)
  - `flavor`: ServiceFlavor name or list of ServiceFlavors. If no SF is specified then all flavors within each site are retrieved. 
  - `site`: Site name or list of sites. If no site is specified then service flavor results are returned for all sites

#### Response

Headers: `Status: 200 OK`

##### Response body 

    <root>
      <Profile name="A_POEM-PROFILE">
        <Site Site="A_SITE-NAME">
          <Flavor Flavor="A_FLAVOR">
            <Availability timestamp="2015-03" availability="99.99999900000002" reliability="99.99999900000002"/>
          </Flavor>
          <Flavor Flavor="B_FLAVOR">
            <Availability timestamp="2015-03" availability="99.99999900000002" reliability="99.99999900000002"/>
          </Flavor>
          .
          .
          <Flavor Flavor="N_FLAVOR">
            <Availability timestamp="2015-03" availability="99.99999900000002" reliability="99.99999900000002"/>
          </Flavor>
        </Site>
      </Profile>
    </root>


### Availability Profiles

#### Input

    /AP

- mandatory
  - `name`: Profile name (both name and namespace are needed)
  - `namespace`: Profile namespace (both name and namespace are needed)


#### Response

Headers: `Status: 200 OK`

##### Response body

    <root>
      <profile id="some_id" name="name" namespace="namespace" poems="poem_name">
        <AND>
          <OR>
            <Group service_flavor="A_FLAVOR"/>
            <Group service_flavor="B_FLAVOR"/>
            <Group service_flavor="C_FLAVOR"/>
            <Group service_flavor="D_FLAVOR"/>
            <Group service_flavor="E_FLAVOR"/>
          </OR>
          <OR>
            <Group service_flavor="F_FLAVOR"/>
            <Group service_flavor="G_FLAVOR"/>
          </OR>
          <OR>
            <Group service_flavor="H_FLAVOR"/>
          </OR>
        </AND>
      </profile>
      .
      .
    </root>


### Recalculation requests

#### Input

    /recomputations


### List POEM profiles

#### Input

    /poems


### Service Metric Status results

#### Input

    /status/timeline/metric?[start_time]&[end_time]&[name]&[host]&[flavor]&[vo]&[profile]

- mandatory:
  - `start_time`: UTC time in W3C format
  - `end_time`: UTC time in W3C format
  - `name`: Metric name
  - `host`: service host fqdn
  - `flavor`: service flavor name
  - `vo`: vo name
  - `profile`: POEM profile name

#### Response

Headers: `Status: 200 OK`

##### Response body

    xml text

### Service Endpoint Status results

#### Input

    /status/timeline/endpoint?[start_time]&[end_time]&[host]&[flavor]&[vo]&[profile]

- mandatory:
  - `start_time`: UTC time in W3C format
  - `end_time`: UTC time in W3C format
  - `host`: service host fqdn
  - `flavor`: service flavor name
  - `vo`: vo name
  - `profile`: POEM profile name

#### Response

Headers: `Status: 200 OK`

##### Response body

    xml text


### Service Flavor Status results

#### Input

    /status/timeline/flavor?[start_time]&[end_time]&[site]&[flavor]&[vo]&[profile]

- mandatory:
  - `start_time`: UTC time in W3C format
  - `end_time`: UTC time in W3C format
  - `site`: site name
  - `flavor`: service flavor name
  - `vo`: vo name
  - `profile`: POEM profile name

#### Response

Headers: `Status: 200 OK`

##### Response body

    xml text



### Site Status results

#### Input

    /status/timeline/site?[start_time]&[end_time]&[site]&[vo]&[profile]

- mandatory:
  - `start_time`: UTC time in W3C format
  - `end_time`: UTC time in W3C format
  - `site`: site name
  - `vo`: vo name
  - `profile`: POEM profile name

#### Response

Headers: `Status: 200 OK`

##### Response body

    xml text


### Bulk Status results

#### Input

    /status/metrics/timeline/<group>[/<metric>]


#### Response

Headers: `Status: 200 OK`

##### Response body

    xml text



## POST methods

### Availability profiles