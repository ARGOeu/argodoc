---
title: Web UI | ARGO
---

## Description
# Web UI module for the ARGO Framework

* based on Lavoisier Framework - http://software.in2p3.fr/lavoisier
* prerequisites : a server certificate and java


## Installation 
# the $HOME_LAVOISIER is the directory where do you install the service
* cd $HOME_LAVOISIER
* wget https://github.com/ARGOeu/argo-egi-web/archive/master.zip
* unzip master.zip
* mkdir certificate
* put your certificate in p12 format in this directory


## Configuration 
# edit lavoisier-hidden.properties
* cd $HOME_LAVOISIER/argo-egi-web-master/etc
* vi lavoisier-hidden.properties
* complete certificate.password and certificate.password
*

# edit lavoisier-service.properties
* cd $HOME_LAVOISIER/argo-egi-web-master/etc
* vi lavoisier-service.properties
* lavoisier.ssl.trustStore=path_to_your_ca  (/etc/grid-security/certificates)
* lavoisier.ssl.keyStore= path to the lavoisier certificate
* lavoisier.ssl.keyStorePassword= password of this certificate

## Examples

# Start the service 

* ./bin/lavoisier.sh console
* check the logs and access to the interface : http://yourmachine:8080/lavoisier
* if everything goes well 
*  ./bin/lavoisier.sh restart


# Useful commands and hints

* To change the PI base url : vi $HOME_LAVOISIER/etc/ARGO_UI/lavoisier-config.properties
* to stop lavoisier : 
cd $HOME_LAVOISIER
./bin/lavoisier.sh stop


## Links and further reading

* http://software.in2p3.fr/lavoisier/
