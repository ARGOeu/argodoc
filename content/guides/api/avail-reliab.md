---
title: API documentation | ARGO
page_title: API - Availabilities & Reliabilities
font_title: 'fa fa-cogs'
description: API calls for retrieving computed Availabilities and Reliabilities
---

## API Data

| Base URL | <code>https://host:port/api/v1/</code> |
| **Default Port**         | <code>443</code>  |
| **Central instance of the ARGO production service** |  <code>https://snf-629551.vm.okeanos.grnet.gr/api/v1/</code> |

## API Calls

| Name  | Description | Shortcut |
| GET: List Availabilities and Reliabilities for Groups | This method can be used to obtain Availability and Reliablity metrics per group type elements (i.e. Sites, NGIs etc). Results can be retrieved on daily or monthly granularity.  |<a href="#1"> Description</a>|
| GET: List Availabilities and Reliabilities for Service Flavors | This method can be used to obtain Availability and Reliability metrics for Service Flavors per Site. Results can be retrieved on daily or monthly granularity. | <a href="#2"> Description</a>|

<a id="1"></a>

## GET: List Availabilities and Reliabilities for Groups

This method can be used to obtain Availability and Reliablity metrics per group type elements (i.e. Sites, NGIs etc). Results can be retrieved on daily or monthly granularity. 

### Input

    /group_availability?[start_time]&[end_time]&[availability_profile]&[group_type]&[granularity]&[infrastructure]&[production]&[monitored]&[certification]&[format]&[group_name]

### Parameters

| Type | Description | Required | Default value |
|`start_time`| UTC time in W3C format| YES ||
|`end_time`| UTC time in W3C format| YES| |
|`availability_profile`| Name of the high level profile concatenated with the profile namespace. Each availability profile matches one POEM profile.| YES| |
|`group_type`|  Valid values are: `site`, `ngi` or `vo` | YES| |
|`granularity`| Possible values: `DAILY`, `MONTHLY` | NO| `DAILY`|
|`infrastructure`| Filter results based on the name of the infrastructure the site belongs to | NO| `Production` |
|`production`| Filter results based on whether they are production sites or not. Possible values: `true`, `false` | NO| `true` |
|`monitored`| Filter results based on whether they are monitored sites or not. Possible values: `true`, `false`. | NO| `true` |
|`certification`| Filter results based on the certification status of each site  | NO| `Certified` |
|`format`| Only xml available right now, so the parameter is void thus deactivated for the time being  | NO| `XML` |
|`group_name`| Site, NGI or VO name. If no name is specified then all sites, NGIs or VOs are retrieved. |NO| |


### Response

Headers: `Status: 200 OK`

#### Response body for `group_type=site`

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

#### Response body for `group_type=ngi`

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


