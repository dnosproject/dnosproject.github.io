---
title: Host Service (Java version)
permalink: /docs/hostservice-java/
---

## Usage of Host Service (Java version)

DNOS provides a HostService that can be used to retrieve  hosts releated information. In this tutorial we will show you how you can use it to print such information. 

1. Follow the installation instructions [here](https://dnosproject.github.io/docs/home/) to prepare your setup ready.

2. Clone the repository of the sample applications:
```console
$ git clone https://github.com/dnosproject/dnos-apps-java.git
```

3. Run a mininet emulation scenario as follows:
```console
   $ sudo mn --topo=linear,4 --controller=remote,ip=127.0.0.1
```
4. Run the **samplehostservice** app under **dnos-apps-java** directory as follows:
```console
bazel run samplehostservice:samplehostservice_image
```
The above command creates an image of the application and run it in a docker container. You should be able to see it by running the following command: 
```console
 $ docker ps
```

5. After running the above application, you will see an output like this:
```console
2019-02-22 23:13:58 INFO  samplehostservice:93 - Number of Hosts:4
2019-02-22 23:13:58 INFO  samplehostservice:123 - of:0000000000000003:F2:FD:27:C7:FE:90
2019-02-22 23:13:58 INFO  samplehostservice:75 - 10.0.0.3;F2:FD:27:C7:FE:90;of:0000000000000003:1
2019-02-22 23:13:58 INFO  samplehostservice:75 - 10.0.0.4;56:14:2F:26:0C:DD;of:0000000000000004:1
2019-02-22 23:13:58 INFO  samplehostservice:75 - 10.0.0.2;A2:15:4A:A2:8E:EA;of:0000000000000002:1
2019-02-22 23:13:58 INFO  samplehostservice:75 - 10.0.0.1;FE:30:5B:4B:35:17;of:0000000000000001:1
2019-02-22 23:13:58 INFO  samplehostservice:123 - of:0000000000000004:56:14:2F:26:0C:DD
2019-02-22 23:13:58 INFO  samplehostservice:123 - of:0000000000000001:FE:30:5B:4B:35:17
2019-02-22 23:13:58 INFO  samplehostservice:123 - of:0000000000000002:A2:15:4A:A2:8E:EA
```

## How does the samplehostservice application work? 

1. First, we need to create a gRPC channel, a HostService gRPC stub, and a TopologyService gRPC stub. We also use an external Conguration service which is part of [dnos-services](https://github.com/dnosproject/dnos-services.git) repository to initalize log4j. 
```java
ManagedChannel channel;
    String controllerIP;
    String grpcPort;

    // Initialize a remote config service for logging
    ConfigService configService = new ConfigService();
    configService.init();
    controllerIP = configService.getConfig().getControllerIp();
    grpcPort = configService.getConfig().getGrpcPort();
    HostServiceStub hostServiceStub;
    TopoServiceStub topologyServiceStub;



    // Creates a gRPC channel
    channel =
        ManagedChannelBuilder
                .forAddress(controllerIP, Integer.parseInt(grpcPort))
                .usePlaintext()
                .build();

    // Creates hostService and topoService stubs and assigns them to the gRPC channel.

    hostServiceStub = HostServiceGrpc.newStub(channel);
    topologyServiceStub = TopoServiceGrpc.newStub(channel);
```

2. Second, we call HostService and TopologyService functions to use them for printing of host related information: 
```java
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

      // Retrieves topology graph and prints the list of hosts connected to each device.
      topologyServiceStub.getGraph(empty,
              new StreamObserver<TopologyGraphProto>() {
          @Override
          public void onNext(TopologyGraphProto value) {


              for(TopologyVertexProto topologyVertexProto: value.getVertexesList()) {

                  DeviceIdProto deviceIdProto = DeviceIdProto
                          .newBuilder()
                          .setDeviceId(topologyVertexProto.getDeviceId().getDeviceId())
                          .build();

                  hostServiceStub.getConnectedHostsByDeviceId(deviceIdProto,
                          new StreamObserver<Hosts>() {
                              @Override
                              public void onNext(Hosts value) {
                                  List<HostProto> hostProtoList = value.getHostList();
                                  for(HostProto hostProto: hostProtoList) {
                                      log.info(deviceIdProto.getDeviceId()
                                              + ":" + hostProto.getHostId().getMac());
                                  }
                              }

                              @Override
                              public void onError(Throwable t) {}

                              @Override
                              public void onCompleted() {}
                          });
              }

          }

          @Override
          public void onError(Throwable t) {}

          @Override
          public void onCompleted() {}
      });
```

