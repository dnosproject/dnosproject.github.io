---
title : 
permalink: /docs/home/
redirect_from: /docs/index.html
---

## Getting started

1. First, you need to install ONOS using the instructions that are posted here: [Developer Quick Start](https://wiki.onosproject.org/display/ONOS/Developer+Quick+Start)

2. Second, you need to install docker on your machine using the instructions that are posted here: [Docker Installation Guideline](https://docs.docker.com/install/)

3. Third, you need to download the binary version of DNOS to install on ONOS as an application. You can find the latest releases here: [DNOS Releases](https://github.com/dnosproject/grpc-integration/releases)

4. Run ONOS as explained in the first step and install DNOS as an application using the following command: 
```console
$ onos-app IP_OF_YOUR_MACHINE install onos-apps-grpc-integration-oar.oar
```
5. Activate the app in ONOS using the following command: 
```console
$ app activate org.onosproject.grpc-integration. 
```
It is ready to execute [sample applications]()!     



