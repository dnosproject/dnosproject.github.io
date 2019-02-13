---
title: Topology Service (Java version)
permalink: /docs/toposervice/
---

## Usage of the Topology Service in a Java application

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

5. After running the application, it prints the number of links and topology graph information as follows: 
```console
  2019-02-12 23:46:28 INFO  topogrpc:45 - Number of links:2
  2019-02-12 23:46:28 INFO  topogrpc:60 - of:0000000000000002:2 -->of:0000000000000001:2
  2019-02-12 23:46:29 INFO  topogrpc:60 - of:0000000000000001:2 -->of:0000000000000002:2
``` 

### How does the sample topology service application work? 

1. First, we need to create a gRPC channel and TopologyService gRPC stub. We also use an external Conguration service which is part of [dnos-services](https://github.com/dnosproject/dnos-services.git) repository to initalize log4j. 
```java
ManagedChannel channel;
    String controllerIP;
    String grpcPort;

    ConfigService configService = new ConfigService();
    configService.init();
    controllerIP = configService.getConfig().getControllerIp();
    grpcPort = configService.getConfig().getGrpcPort();
    TopoServiceStub topologyServiceStub;

    channel =
        ManagedChannelBuilder
                .forAddress(controllerIP, Integer.parseInt(grpcPort))
                .usePlaintext()
                .build();

    topologyServiceStub = TopoServiceGrpc.newStub(channel);
``` 

2. Second, we need to create an Empty request that should be sent as an argument when we call topology servcie functions (i.e. currentTopology and getGraph functions). CurrentTopology function returns high level information about topology such as number of links, the time that topology is created, number of clusters in the topology, etc (For more information, please take a look at [Topology class](http://api.onosproject.org/1.2.1/org/onosproject/net/topology/Topology.html) in ONOS. getGraph function returns a graph of the network topology that includes topology edges and vertices.  
```java
    ServicesProto.Empty empty = ServicesProto.Empty.newBuilder().build();

    topologyServiceStub.currentTopology(empty,
            new StreamObserver<TopologyProto>() {
              @Override
              public void onNext(TopologyProto value) {

                  log.info("Number of links:" + value.getLinkCount());
              }

              @Override
              public void onError(Throwable t) {}

              @Override
              public void onCompleted() {}
            });

    topologyServiceStub.getGraph(empty, new StreamObserver<TopologyGraphProto>() {
        @Override
        public void onNext(TopologyGraphProto value) {
            for(TopologyEdgeProto topologyEdgeProto: value.getEdgesList()) {

                log.info(topologyEdgeProto.getLink().getSrc().getDeviceId() +
                        ":" + topologyEdgeProto.getLink().getSrc().getPortNumber() +
                " -->" + topologyEdgeProto.getLink().getDst().getDeviceId() +
                         ":" + topologyEdgeProto.getLink().getDst().getPortNumber());
            }
        }

        @Override
        public void onError(Throwable t) {}

        @Override
        public void onCompleted() {}
    });
```
