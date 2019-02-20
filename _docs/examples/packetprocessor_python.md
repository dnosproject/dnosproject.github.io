---
title: Packet Processor (Python version)
permalink: /docs/packetprocessor-python/ 
---

## A Simple External Packet Processor Using Python

DNOS provides a gRPC based event notification service that can be used in management applications to receive incoming events such as **PACKET_IN** events. In this tutorial we explain how you can 
use that service in a python application to receive incoming packets from ONOS. 

1. Follow the installation instructions [here](https://dnosproject.github.io/docs/home/) to prepare your setup ready.

2. Clone the repository of the sample python applications as follows:
```console
git clone https://github.com/dnosproject/dnos-apps-python.git
```

3. 3. Run a mininet emulation scenario as follows:
```console
   $ sudo mn --topo=linear,2 --controller=remote,ip=127.0.0.1
```

4. Run the **samplepacketprocessor** app under **dnos-apps-python** directory as follows.
```console
bazel run samplepacketprocessor:samplepacketprocessor 
```
5. Run pingall from mininet command line. After running the pingall, the first incoming ICMP packets will be sent to the controller and the event notification service in DNOS application send them to the external sample packet processer application. The application prints an output like this:



   
