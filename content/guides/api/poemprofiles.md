---
title: API documentation | ARGO
page_title: API - POEM Profiles 
font_title: 'fa fa-cogs'
description: API Calls for retrieving available POEM profiles
---
## API Data

| Base URL | <code>https://host:port/api/v1/</code> |
| **Default Port**         | <code>443</code>  |
| **Central instance of the ARGO production service** |  <code>https://snf-629551.vm.okeanos.grnet.gr/api/v1/</code> |

<a id="1"></a>

## GET: List POEM Profiles

This method can be used to retrieve a list of current POEM profiles.

### Input

    /poems

### Response

Headers: `Status: 200 OK`

#### Response body

    <root>
      <Poem profile="ch.cern.sam.CLOUD-MON"/>
      <Poem profile="ch.cern.sam.GLEXEC"/>
      <Poem profile="ch.cern.sam.OPS_MONITOR"/>
      .
      .
    </root>


