---
title: Packet Processor
permalink: /docs/packetprocessor/
---

## A Simple Packet Processor

DNOS provides a gRPC based event notification service that can be used in management applications to receive incoming events such as **PACKET_IN** events. In this tutorial we explain how can you use that service in an application to receive incoming packets from ONOS. 


1. Follow the installation instructions [here](https://dnosproject.github.io/docs/home/) to prepare your setup ready.

2. Clone the repository of the sample applications:
```console
$ git clone https://github.com/dnosproject/dnos-apps.git
```

3. Run a mininet emulation scenario as follows:
```console
   $ sudo mn --topo=linear,2 --controller=remote,ip=127.0.0.1
```

4. Run the **samplepacketprocessor** app under dnos-apps as follows:
```console
   $ bazel run  samplepacketprocessor:samplepacketprocessorgrpc_image
 
```

5. Run pingall from mininet command line. After running the pingall, the first incoming ICMP packets will be sent to the controller and the event notification service in DNOS application send them to the external sample packet processer application.

