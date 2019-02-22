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
$ git clone https://github.com/dnosproject/dnos-apps-java.git
```

3. Run a mininet emulation scenario as follows:
```console
   $ sudo mn --topo=linear,4 --controller=remote,ip=127.0.0.1
```

4. Run the **topogrpc** app under dnos-apps as follows.
```console
   $ bazel run  sampletopology:sampletopology_image 
```
The above command creates an image of the application and run it in a docker container. You should be able to see it by running the following command: 
```console
 $ docker ps
```

5. After running the application, it prints the number of links and topology graph information as follows: 
```console
2019-02-22 01:46:06 INFO  sampletopology:114 - Number of Hosts:4
2019-02-22 01:46:06 INFO  sampletopology:59 - Number of links:6
2019-02-22 01:46:06 INFO  sampletopology:76 - of:0000000000000002:3 -->of:0000000000000003:2
2019-02-22 01:46:06 INFO  sampletopology:76 - of:0000000000000003:3 -->of:0000000000000004:2
2019-02-22 01:46:06 INFO  sampletopology:76 - of:0000000000000002:2 -->of:0000000000000001:2
2019-02-22 01:46:06 INFO  sampletopology:76 - of:0000000000000004:2 -->of:0000000000000003:3
2019-02-22 01:46:06 INFO  sampletopology:76 - of:0000000000000003:2 -->of:0000000000000002:3
2019-02-22 01:46:06 INFO  sampletopology:76 - of:0000000000000001:2 -->of:0000000000000002:2
2019-02-22 01:46:06 INFO  sampletopology:96 - 10.0.0.3;56:F5:08:C3:FA:82;of:0000000000000003:1
2019-02-22 01:46:06 INFO  sampletopology:96 - 10.0.0.4;26:0D:F0:DB:B0:CB;of:0000000000000004:1
2019-02-22 01:46:06 INFO  sampletopology:96 - 10.0.0.1;16:67:FD:86:79:A4;of:0000000000000001:1
2019-02-22 01:46:06 INFO  sampletopology:96 - 10.0.0.2;CE:84:0D:26:D5:CB;of:0000000000000002:1
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
 Empty empty = Empty.newBuilder().build();

    // Retrieves current topology information
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

    // Retrieves topology graph
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
3. We can also retrieve the list of hosts that have beed detected by ONOS controller. For example, we print the ip address, mac address, and the switch and the port number that host is connected to. 
```java
// Retrieves list of hosts in the network topology
 // Retrieves list of hosts in the network topology
    hostServiceStub.getHosts(empty, new StreamObserver<Hosts>() {
        @Override
        public void onNext(Hosts value) {
            for(HostProto hostProto:value.getHostList()) {

                log.info(hostProto.getIpAddresses(0)
                + ";" + hostProto.getHostId().getMac() + ";" +
                hostProto.getLocation().getConnectPoint().getDeviceId()
                + ":" + hostProto.getLocation().getConnectPoint().getPortNumber());
            }
        }

        @Override
        public void onError(Throwable t) {}

        @Override
        public void onCompleted() {}
    });
```
4. We can also retireve the number of hosts that have been detected by ONOS controler. 
```java
// Returns number of hosts in the network topology.
    hostServiceStub.getHostCount(empty, new StreamObserver<HostCountProto>() {
        @Override
        public void onNext(HostCountProto value) {
            log.info("Number of Hosts:" + value.getCount());
        }

        @Override
        public void onError(Throwable t) {}

        @Override
        public void onCompleted() {}
    });
```
