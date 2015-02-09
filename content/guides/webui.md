---
title: Web UI | ARGO
---

# Web UI

This document describes the Web UI installation and configuration process. 


Web UI module for the ARGO Framework :

* based on Lavoisier Framework - http://software.in2p3.fr/lavoisier
* prerequisites : a server certificate and java


Installation - the $HOME_LAVOISIER is the directory where do you install the service
* cd $HOME_LAVOISIER
* wget https://github.com/ARGOeu/argo-ui/archive/master.zip
* unzip master.zip
* mkdir certificate
* put your certificate in p12 format in this directory
* cd $HOME_LAVOISIER/argo-ui-master/etc
* vi lavoisier-hidden.properties
* complete certificate.password and certificate.password


Start the service 

* ./bin/lavoisier.sh console
* check the logs and access to the interface : http://yourmachine:8080/lavoisier
* if everything goes well 
*  ./bin/lavoisier.sh restart
