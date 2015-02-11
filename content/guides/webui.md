---
title: Web UI | ARGO
---

## Web UI Description

This document describes the Web UI installation and configuration process. 

Web UI module for the ARGO Framework :

* based on Lavoisier Framework - `http://software.in2p3.fr/lavoisier`
* prerequisites : a server certificate and java

## Installation 

 `$HOME_LAVOISIER` is the directory where do you install the service

    # cd $HOME_LAVOISIER
    # wget https://github.com/ARGOeu/argo-ui/archive/master.zip
    # unzip master.zip
    # mkdir certificate

Put your certificate in p12 format in this directory

    # cd $HOME_LAVOISIER/argo-ui-master/etc
    # vi lavoisier-hidden.properties
Finally, complete certificate.password and certificate.password
Eventually modify the cache.baseDirectory property - if you want to put it elsewhere



    # cd $HOME_LAVOISIER/argo-ui-master/etc
    # vi lavoisier-service.properties
Complete lavoisier.ssl.trustStore (/etc/grid-security/certificates) , lavoisier.ssl.keyStore (path to p12 certificate) , and lavoisier.ssl.keyStorePassword


Start the service 

- `./bin/lavoisier.sh console`
- check the logs and access to the interface : `http://yourmachine:8080/lavoisier`
- if everything goes well `./bin/lavoisier.sh restart`


 ## Useful commands and hints

To change the PI base url : 

    # vi $HOME_LAVOISIER/etc/ARGO_UI/lavoisier-config.properties
    
To stop lavoisier : 

    # cd $HOME_LAVOISIER
    # ./bin/lavoisier.sh stop
    
To modify the list of administrators 

    # vi $HOME_LAVOISIER/etc/ARGO_UI/resources/xml/administrators.xml
