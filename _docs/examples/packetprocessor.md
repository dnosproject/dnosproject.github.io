---
title: Packet Processor (Java version)
permalink: /docs/packetprocessor/
---

## A Simple External Packet Processor

DNOS provides a gRPC based event notification service that can be used in management applications to receive incoming events such as **PACKET_IN** events. In this tutorial we explain how can you use that service in a java application to receive incoming packets from ONOS. 

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
   $ bazel run  samplepacketprocessor:samplepacketprocessorgrpc_image 
```
The above command creates an image of the application and run it in a docker container. You should be able to see it by running the following command: 
```console
 $ docker ps
```

5. Run pingall from mininet command line. After running the pingall, the first incoming ICMP packets will be sent to the controller and the event notification service in DNOS application send them to the external sample packet processer application. The application prints an output like this:
```console
2019-02-12 16:45:24 INFO  samplepacketprocessor:101 - An IPv4 Packet has been received
2019-02-12 16:45:24 INFO  samplepacketprocessor:101 - An IPv4 Packet has been received
2019-02-12 16:45:24 INFO  samplepacketprocessor:101 - An IPv4 Packet has been received
```

### How does the sample packet processr application work?

1. First of all, we need to create a gRPC channel and an EventNotifcationStub. We also use an external Conguration service which is part of [dnos-services](https://github.com/dnosproject/dnos-services.git) repository to initalize log4j.
```java
    ManagedChannel channel;
    ConfigService configService = new ConfigService();
    configService.init();

    EventNotificationStub packetNotificationStub;

    // Create a managed gRPC channel.
    channel = ManagedChannelBuilder
            .forAddress("127.0.0.1", 50051)
            .usePlaintext()
            .build();

    packetNotificationStub = EventNotificationGrpc.newStub(channel);
```

2. In the second step, we need to register the application in the remote gRPC server which is running in DNOS application as follows: 
```java
RegistrationRequest request = RegistrationRequest
            .newBuilder()
            .setClientId(clientId)
            .build();
    // Registers the client
    packetNotificationStub.register(
        request, new StreamObserver<RegistrationResponse>() {
          @Override
          public void onNext(RegistrationResponse value) {
            serverId = value.getServerId();
          }

          @Override
          public void onError(Throwable t) {}

          @Override
          public void onCompleted() {}
        });
```

3. Finally, we need to create and subscribe to **PACKET_EVENT** topic by calling its remote function:
```java
// Creates a packet event topic
    Topic packettopic =
        Topic.newBuilder()
                .setClientId(clientId)
                .setType(topicType.PACKET_EVENT)
                .build();

    // Implements a packet processor
    class PacketEvent implements Runnable {

      @Override
      public void run() {

        packetNotificationStub.onEvent(
            packettopic, new StreamObserver<Notification>() {
              @Override
              public void onNext(Notification value) {

                PacketContextProto packetContextProto = value.getPacketContext();

                PacketContextProto finalPacketContextProto = packetContextProto;
                byte[] packetByteArray =
                    finalPacketContextProto.getInboundPacket().getData().toByteArray();
                Ethernet eth = new Ethernet();

                try {
                  eth =
                      Ethernet.deserializer()
                          .deserialize(packetByteArray, 0, packetByteArray.length);
                } catch (DeserializationException e) {
                  e.printStackTrace();
                }

                if (eth == null) {
                  return;
                }

                long type = eth.getEtherType();
                if(Ethernet.TYPE_IPV4 == type) {
                    log.info("An IPv4 Packet has been received");
                }

              }

              @Override
              public void onError(Throwable t) {}

              @Override
              public void onCompleted() {
                log.info("completed");
              }
            });

        while (true) {}
      }
    }

    PacketEvent packetEvent = new PacketEvent();
    Thread t = new Thread(packetEvent);
    t.start();
```
