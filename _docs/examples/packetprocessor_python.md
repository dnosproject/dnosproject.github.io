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

3. Run a mininet emulation scenario as follows:
```console
   $ sudo mn --topo=linear,2 --controller=remote,ip=127.0.0.1
```

4. Run the **samplepacketprocessor** app under **dnos-apps-python** directory as follows.
```console
bazel run samplepacketprocessor:samplepacketprocessor 
```
5. Run pingall from mininet command line. After running the pingall, the first incoming ICMP packets will be sent to the controller and the event notification service in DNOS application send them to the external sample packet processer application. The application prints an output like this:
```console
An IPv4 packet has been received
An IPv4 packet has been received
An IPv4 packet has been received
An IPv4 packet has been received
```

## How does the sample packet processor application work? 

1. First of all, we need to create a gRPC channel and register the client in the remote gRPC server which is running in DNOS application as follows:
```python
     channel = grpc.insecure_channel('localhost:50051')
     eventNotificationStub = ServicesProto_pb2_grpc.EventNotificationStub(channel)

     request = EventNotificationProto_pb2.RegistrationRequest(clientId = "packet_processor_python")
     topic = EventNotificationProto_pb2.Topic(clientId = "packet_processor_python"
                                              , type = 0)

     # Register to PACKET_EVENT
     response = eventNotificationStub.register(request)
```

2. Second, we need subcribe to **PACKET_EVENT** to receive incoming packets from DNOS. Then we will need to go through the list of incoming events (in this example packet events) and process incoming packets using a third-party library to extract its information. In this example, we want to print a message whenever we receive an IPv4 packet. 
```python
eventObserver = eventNotificationStub.onEvent(topic)
for event in eventObserver:
          pktContext = event.packetContext
          if pktContext is None:
              return
          inboundPkt = pktContext.inboundPacket
          pkt = packet.Packet(inboundPkt.data)
          for p in pkt:
              if type(p)!= str:
                 if p.protocol_name == "ipv4":
                     print("An IPv4 packet has been received")
```
   
