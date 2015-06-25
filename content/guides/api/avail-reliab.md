---
title: API documentation | ARGO
page_title: API - Availabilities & Reliabilities
font_title: 'fa fa-cogs'
description: API calls for retrieving computed Availabilities and Reliabilities
---

## API Calls

| Name  | Description | Shortcut |
| GET: List Availabilities and Reliabilities for Groups | The following methods can be used to obtain Availability and Reliablity metrics per group type elements (i.e. Endpoint Groups, Group of Endpoint Groups etc). Results can be retrieved on daily or monthly granularity.  |<a href="#1"> Description</a>|
| GET: List Availabilities and Reliabilities for Service Flavors | This method can be used to obtain Availability and Reliability metrics for Service Flavors per Site. Results can be retrieved on daily or monthly granularity. | <a href="#2"> Description</a>|

<a id="1"></a>

## GET: List Availabilities and Reliabilities for Endpoint Groups, Group of Endpoint Groups

The following methods can be used to obtain Availability and Reliability metrics per group type elements (i.e. Endpoint Groups, Group of Endpoint Groups etc). Results can be retrieved on daily or monthly granularity.

### Input

Endpoint Groups
    /endpoint_group_availability?[start_time]&[end_time]&[job]&[granularity]&[format]&[group_name]&[supergroup_name]

### Parameters

| Type | Description | Required | Default value |
|`start_time`| UTC time in W3C format| YES ||
|`end_time`| UTC time in W3C format| YES| |
|`job`| Name of the job that contains all the information about the profile, filter tags etc. | YES| |
|`format`| Only xml available right now, so the parameter is void thus deactivated for the time being  | NO| `XML` |
|`group_name`| Name of the Endpoint Groups. If no name is specified then all Endpoint Groups are retrieved. |NO| |
|`supergroup_name`| Name of the group that groups the Endpoint Groups. If no name is specified then all groups are retrieved. |NO| |

### Input

Group of Endpoint Groups
    /group_groups_availability?[start_time]&[end_time]&[job]&[granularity]&[format]&[group_name]

### Parameters

| Type | Description | Required | Default value |
|`start_time`| UTC time in W3C format| YES ||
|`end_time`| UTC time in W3C format| YES| |
|`job`| Name of the job that contains all the information about the profile, filter tags etc. | YES| |
|`format`| Only xml available right now, so the parameter is void thus deactivated for the time being  | NO| `XML` |
|`group_name`| Name of the group that groups the Endpoint Groups. If no name is specified then all groups are retrieved. |NO| |


### Response

Headers: `Status: 200 OK`

#### Response body for `/endpoint_group_availability` API call

    <root>
      <Job name="Job_A">
        <EndpointGroup name="Site-Name" SuperGroup="SuperGroup-A">
          <Availability timestamp="YYYY-MM-DD" availability="1" reliability="1"/>
          <Availability timestamp="YYYY-MM-DD" availability="1" reliability="1"/>
        </EndpointGroup>
        <EndpointGroup name="Site-Name-Another" SuperGroup="SuperGroup-B">
          <Availability timestamp="YYYY-MM-DD" availability="1" reliability="1"/>
          <Availability timestamp="YYYY-MM-DD" availability="1" reliability="1"/>
        </EndpointGroup>
        .
        .
        .
      </Job>
    </root>

#### Response body for `/group_groups_availability` API call

    <root>
      <Job name="Job_A">
        <SuperGroup name="GROUP_A">
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
        </SuperGroup>
      </Job>
    </root>


#### Response body for `group_type=vo`

     <root>
       <Profile name="A_PROFILE">
         <Vo VO="ops">
           <Availability timestamp="2013-11-09" availability="100" reliability="100"></Availability>
           <Availability timestamp="2013-11-19" availability="50" reliability="100"></Availability>
           <Availability timestamp="2013-12-09" availability="100" reliability="100"></Availability>
         </Vo>
       </Profile>
     </root>

<a id="2"></a>

## GET: List Availabilities and Reliabilities for Service Flavors

This method can be used to obtain Availability and Reliability metrics for Service Flavors per Site. Results can be retrieved on daily or monthly granularity.

### Input

    /service_flavor_availability?[start_time]&[end_time]&[profile]&[granularity]&[format]&[flavor]&[site]


### Parameters

| Type | Description | Required | Default value |
|`start_time`| UTC time in W3C format| YES ||
|`end_time`| UTC time in W3C format| YES| |
|`profile`|  Name of the POEM profile according to which the result was calculated. | YES| |
|`granularity`| Possible values: `DAILY`, `MONTHLY` | NO| `DAILY`|
|`format`| Only xml available right now, so the parameter is void thus deactivated for the time being  | NO| `XML` |
|`flavor`| Service Flavor name or list of Service Flavors. If no SF is specified then all flavors within each site are retrieved.  | NO| `all flavors` |
|`site`|  Site name or list of sites. If no site is specified then service flavor results are returned for all sites  | NO| `all sites` |

### Response

Headers: `Status: 200 OK`

#### Response body

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
