---
title: Topology Service (Java version)
permalink: /docs/toposervice/
---

## A Sample Topology Service

DNOS provides a gRPC based topology service that can be used in management applications 
to retrieve topology information. In this tutorial we will show you how can use that to print topology graph. 

1. Follow the installation instructions [here](https://dnosproject.github.io/docs/home/) to prepare your setup ready.

2. Clone the repository of the sample applications:
```console
$ git clone https://github.com/dnosproject/dnos-apps.git
```

3. Run a mininet emulation scenario as follows:
```console
   $ sudo mn --topo=linear,2 --controller=remote,ip=127.0.0.1
```

4. Run the **samplepacketprocessor** app under dnos-apps as follows.
```console
   $ bazel run  topogrpc:topogrpc_image 
```
The above command creates an image of the application and run it in a docker container. You should be able to see it by running the following command: 
```console
 $ docker ps
```

5. After running the application, the application prints the number of links and topology graph information as follows: 
```console
  2019-02-12 23:46:28 INFO  topogrpc:45 - Number of links:2
  2019-02-12 23:46:28 INFO  topogrpc:60 - of:0000000000000002:2 -->of:0000000000000001:2
  2019-02-12 23:46:29 INFO  topogrpc:60 - of:0000000000000001:2 -->of:0000000000000002:2
``` 
