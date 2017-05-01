# Application Service Element

__[Hands On 1](#hands-on-1)__ demonstrates how to to setup two BACnet/IT Stacks on localhost with each two BACnet devices. A device from stack 1 sends a ReadPropertyRequest to a device from stack 2. [BACnet4J](https://github.com/empeeoh/BACnet4J) is used to get the byte array of the ReadPropertyRequest. The destinations address is already resolved (localhost), nevertheless the usage of a directory binding is shown.

__[Hands On 2](#hands-on-2)__ demonstrates how to setup one BACnet/IT Stack on localhost with two BACnet devices. One BACnet device sends a WhoIsRequest to the other device. The WhoIsRequest is represented as a byte array, therefore no dependencies to other projects like BACnet4J (to represent BACnet service) is needed. The destinations address is already resolved (localhost), nevertheless the usage of a directory binding is shown.


__[Hands On 3](#hands-on-3)__ demonstrates how to run a BACnet/IT Stack with more than one transport binding.



# Hands on 1

## Download
1. Create a new empty directory __BACnetIT__ and make it the current directory
2. Download the source code of projects __ase__, __transport-binding-ws__, __directory-binding-dnssd__ and __samples-and-tests__  
__ase__ project:  
```git clone https://github.com/fhnw-BACnet-IT/ase.git```  
__transport-binding-ws__ project:  
```git clone https://github.com/fhnw-BACnet-IT/transport-binding-ws.git```  
__directory-binding-dnssd__ project:  
```git clone https://github.com/fhnw-BACnet-IT/directory-binding-dnssd.git```  
__samples-and-tests__ project:  
```git clone https://github.com/fhnw-BACnet-IT/samples-and-tests.git```  



## Build
1. Make __BACnetIT/samples-and-tests__ the current directory.
2. Note that project __samples-and-tests__ has dependencies to projects __ase__, __transport-binding-ws__ and __directory-binding-dnssd__ so ensure that all projects are stored at the same level in the __BACnetIT__ folder.
3. Build __samples-and-tests__ using Gradle Wrapper:  
```
  ./gradlew clean build -x test
```  
4. Find all needed dependencies as jar files under __samples-and-tests/build/distributions__


## Setup

### Description / Story:  
Hands On 1 demonstrates how to to setup two BACnet/IT Stacks on localhost with each two BACnet devices. A device from stack 1 sends a ReadPropertyRequest to a device from stack 2. BACnet4J (https://github.com/empeeoh/BACnet4J) is used to get the byte array of the ReadPropertyRequest. The destinations address is already resolved (localhost), nevertheless the usage of a directory binding is shown.

### Preparation
Ensure the builded jars are in java class path.

#### Setup stack 1 on localhost at port 8080

Create an instance of the ConnectionFactory class

```java
ConnectionFactory connectionFactory1 = new ConnectionFactory();
```  
Add a Transport Binding for outgoing and incoming communication  

```java
int port1 = 8080;
connectionFactory1.addConnectionClient("ws", new WSConnectionClientFactory());
connectionFactory1.addConnectionServer("ws", new WSConnectionServerFactory(port1));
```
Create an instance of the Channel class. Segregate the functionality by casting __channel1__ to the interfaces __ChannelConfiguration__ and __ApplicationService__.

```java
Channel channel1 = new Channel();
ChannelConfiguration channelConfiguration1 = (ChannelConfiguration)channel1;
ApplicationService applicationService1 = (ApplicationService)channel1;
```
Implement the ChannelListener interface for each simulated BACnet device and the network port object. Pass a keystore configuration to the network port object to identify the application.

```java
final BACnetEID device1inStack1 = new BACnetEID(1001);
final BACnetEID device2inStack1 = new BACnetEID(1002);
final KeystoreConfig keystoreConfig1 = new KeystoreConfig("dummyKeystores/keyStoreDev1.jks","123456", "operationaldevcert");

final NetworkPortObj npo1 = new NetworkPortObj("ws", 8080, keystoreConfig1);

channelConfiguration1.registerChannelListener(new ChannelListener(device1inStack1) {
    @Override
    public void onIndication(
            final T_UnitDataIndication tUnitDataIndication,
            final ChannelHandlerContext ctx) {
        System.out.println(this.eid.getIdentifierAsString()
                + " got an indication" + tUnitDataIndication.getData());
    }

    @Override
    public void onError(final String cause) {
        System.err.println(cause);
    }

    
});

channelConfiguration1.registerChannelListener(new ChannelListener(device2inStack1) {
    @Override
    public void onIndication(
            final T_UnitDataIndication tUnitDataIndication,
            final ChannelHandlerContext ctx) {
        System.out.println(this.eid.getIdentifierAsString()
                + " got an indication" + tUnitDataIndication.getData());
    }

    @Override
    public void onError(final String cause) {
        System.err.println(cause);
    }

   
});
```

Implement the BACnetEntityListener interface to handle ControlMessages on application level

```java
final BACnetEntityListener bacNetEntityHandler = new BACnetEntityListener() {

            @Override
            public void onRemoteAdded(final BACnetEID eid,final URI remoteUri) {
                DirectoryService.getInstance().register(eid, remoteUri, false, true);
            }

            @Override
            public void onRemoteRemove(final BACnetEID eid) {
                // TODO Auto-generated method stub
            }

            @Override
            public void onLocalRequested(final BACnetEID eid) {
                // TODO Auto-generated method stub
            }

        };
channelConfiguration1.setEntityListener(bacNetEntityHandler);
```

Start the channel passing the connection factory instance containing the transport bindings

```java
channelConfiguration1.initializeAndStart(connectionFactory1);
```


#### Setup stack 2 on localhost at port 9090

Create an instance of the ConnectionFactory class

```java
ConnectionFactory connectionFactory2 = new ConnectionFactory();
```  
Add a Transport Binding for outgoing and incoming communincation  

```java
int port2 = 9090;
connectionFactory2.addConnectionClient("ws", new WSConnectionClientFactory());
connectionFactory2.addConnectionServer("ws", new WSConnectionServerFactory(port2));
```
Create an instance of the Channel class. Segregate the functionality by casting channel1 to the interfaces ChannelConfiguration and ApplicationService.

```java
final Channel channel2 = new Channel();
final ChannelConfiguration channelConfiguration2 = (ChannelConfiguration)channel2;
// final ApplicationService applicationService2 = (ApplicationService)channel2;
```
Implement the ChannelListener interface for each simulated BACnet device and the stacks network port object. Passing a keystore configuration to the network port object to identify the stack.

```java
final BACnetEID device1inStack2 = new BACnetEID(2001);
final BACnetEID device2inStack2 = new BACnetEID(2002);
final KeystoreConfig keystoreConfig2 = new KeystoreConfig("dummyKeystores/keyStoreDev1.jks","123456", "operationaldevcert");

final NetworkPortObj npo2 = new NetworkPortObj("ws", 9090, keystoreConfig2);

channelConfiguration2.registerChannelListener(new ChannelListener(device1inStack2) {
    @Override
    public void onIndication(
            final T_UnitDataIndication tUnitDataIndication,final ChannelHandlerContext ctx) {
        System.out.println(this.eid.getIdentifierAsString()
                + " got an indication" + tUnitDataIndication.getData());
    }

    @Override
    public void onError(final String cause) {
        System.err.println(cause);
    }  
});

channelConfiguration2.registerChannelListener(new ChannelListener(device2inStack2) {
    @Override
    public void onIndication(
            final T_UnitDataIndication tUnitDataIndication,
            final ChannelHandlerContext ctx) {
        System.out.println(this.eid.getIdentifierAsString()
                + " got an indication" + tUnitDataIndication.getData());
    }

    @Override
    public void onError(final String cause) {
        System.err.println(cause);
    }   
});
```

Implement the BACnetEntityListener interface to handle Control Messages on application level

```java
final BACnetEntityListener bacNetEntityHandler2 = new BACnetEntityListener() {

    @Override
    public void onRemoteAdded(final BACnetEID eid, final URI remoteUri) {
        DirectoryService.getInstance().register(eid, remoteUri, false, true);
    }

    @Override
    public void onRemoteRemove(final BACnetEID eid) {
        // TODO Auto-generated method stub
    }

    @Override
    public void onLocalRequested(final BACnetEID eid) {
        // TODO Auto-generated method stub
    }

};
channelConfiguration2.setEntityListener(bacNetEntityHandler2);
```

Start the channel passing the connection factory instance containing transport bindings

```java
channelConfiguration2.initializeAndStart(connectionFactory2);
```

#### Start the directory service
```java
final DiscoveryConfig ds = new DiscoveryConfig(
                "DNSSD", "[DNS IP]",
                "itb.bacnet.ch.", "bds._sub._bacnet._tcp.",
                "dev._sub._bacnet._tcp.", "obj._sub._bacnet._tcp.", false);

try {
                DirectoryService.init();
                DirectoryService.getInstance().setDns(ds);

} catch (final Exception e1) {
                // TODO Auto-generated catch block
                e1.printStackTrace();
}
```


#### Send a ReadPropertyRequest from device1inStack1 to device2inStack2

Create an instance of ReadProperty class

```java
ReadPropertyRequest readRequest = new ReadPropertyRequest(
            new BACnetObjectIdentifier(BACnetObjectType.analogValue, 1),
            BACnetPropertyIdentifier.presentValue
);
```
Got the byte sequence from the confirmed BACnet Service (readRequest)

```java
ByteQueue byteQueue = new ByteQueue();
readRequest.write(byteQueue);
```
Create an instance of TPDU class

```java
TPDU tpdu = new TPDU(device1inStack1, device2inStack2, byteQueue.popAll());
```

Create an instance of T_UnitDataRequest class and pass the tpdu

```java
T_UnitDataRequest unitDataRequest = new T_UnitDataRequest(new URI("ws://localhost:9090"), tpdu, 1, true, null);
```

Pass the unitDataRequest down to the channel

```Java
applicationService1.doRequest(unitDataRequest);
```

device2inStack2 should get an indication from device1inStack1.


#### Complete code example

```java
final ConnectionFactory connectionFactory1 = new ConnectionFactory();

final int port1 = 8080;
connectionFactory1.addConnectionClient("ws",
        new WSConnectionClientFactory());
connectionFactory1.addConnectionServer("ws",
        new WSConnectionServerFactory(port1));
    
final Channel channel1 = new Channel();
final ChannelConfiguration channelConfiguration1 = (ChannelConfiguration)channel1;
final ApplicationService applicationService1 = (ApplicationService)channel1;
    

final BACnetEID device1inStack1 = new BACnetEID(1001);
final BACnetEID device2inStack1 = new BACnetEID(1002);
final KeystoreConfig keystoreConfig1 = new KeystoreConfig("dummyKeystores/keyStoreDev1.jks","123456", "operationaldevcert");

final NetworkPortObj npo1 = new NetworkPortObj("ws", 8080, keystoreConfig1);

channelConfiguration1.registerChannelListener(new ChannelListener(device1inStack1) {
    @Override
    public void onIndication(
            final T_UnitDataIndication tUnitDataIndication,
            final ChannelHandlerContext ctx) {
        System.out.println(this.eid.getIdentifierAsString()
                + " got an indication" + tUnitDataIndication.getData());
    }

    @Override
    public void onError(final String cause) {
        System.err.println(cause);
    }

    
});

channelConfiguration1.registerChannelListener(new ChannelListener(device2inStack1) {
    @Override
    public void onIndication(
            final T_UnitDataIndication tUnitDataIndication,
            final ChannelHandlerContext ctx) {
        System.out.println(this.eid.getIdentifierAsString()
                + " got an indication" + tUnitDataIndication.getData());
    }

    @Override
    public void onError(final String cause) {
        System.err.println(cause);
    }

   
});

final BACnetEntityListener bacNetEntityHandler = new BACnetEntityListener() {

    @Override
    public void onRemoteAdded(final BACnetEID eid,
            final URI remoteUri) {
        DirectoryService.getInstance().register(eid, remoteUri, false,
                true);
    }

    @Override
    public void onRemoteRemove(final BACnetEID eid) {
        // TODO Auto-generated method stub
    }

    @Override
    public void onLocalRequested(final BACnetEID eid) {
        // TODO Auto-generated method stub
    }

};
channelConfiguration1.setEntityListener(bacNetEntityHandler);

channelConfiguration1.initializeAndStart(connectionFactory1);

final ConnectionFactory connectionFactory2 = new ConnectionFactory();

final int port2 = 9090;
connectionFactory2.addConnectionClient("ws",
        new WSConnectionClientFactory());
connectionFactory2.addConnectionServer("ws",
        new WSConnectionServerFactory(port2));

final Channel channel2 = new Channel();
final ChannelConfiguration channelConfiguration2 = (ChannelConfiguration)channel2;
// final ApplicationService applicationService2 = (ApplicationService)channel2;
    

final BACnetEID device1inStack2 = new BACnetEID(2001);
final BACnetEID device2inStack2 = new BACnetEID(2002);
final KeystoreConfig keystoreConfig2 = new KeystoreConfig("dummyKeystores/keyStoreDev1.jks","123456", "operationaldevcert");

final NetworkPortObj npo2 = new NetworkPortObj("ws", 9090, keystoreConfig2);

channelConfiguration2.registerChannelListener(new ChannelListener(device1inStack2) {
    @Override
    public void onIndication(
            final T_UnitDataIndication tUnitDataIndication,
            final ChannelHandlerContext ctx) {
        System.out.println(this.eid.getIdentifierAsString()
                + " got an indication" + tUnitDataIndication.getData());
    }

    @Override
    public void onError(final String cause) {
        System.err.println(cause);
    }

    
});

channelConfiguration2.registerChannelListener(new ChannelListener(device2inStack2) {
    @Override
    public void onIndication(
            final T_UnitDataIndication tUnitDataIndication,
            final ChannelHandlerContext ctx) {
        System.out.println(this.eid.getIdentifierAsString()
                + " got an indication" + tUnitDataIndication.getData());
    }

    @Override
    public void onError(final String cause) {
        System.err.println(cause);
    }

   
});

final BACnetEntityListener bacNetEntityHandler2 = new BACnetEntityListener() {

    @Override
    public void onRemoteAdded(final BACnetEID eid,
            final URI remoteUri) {
        DirectoryService.getInstance().register(eid, remoteUri, false,
                true);
    }

    @Override
    public void onRemoteRemove(final BACnetEID eid) {
        // TODO Auto-generated method stub
    }

    @Override
    public void onLocalRequested(final BACnetEID eid) {
        // TODO Auto-generated method stub
    }

};
channelConfiguration2.setEntityListener(bacNetEntityHandler2);

channelConfiguration2.initializeAndStart(connectionFactory2);

final DiscoveryConfig ds = new DiscoveryConfig(
        "DNSSD", "[DNS IP]",
        "itb.bacnet.ch.", "bds._sub._bacnet._tcp.",
        "dev._sub._bacnet._tcp.", "obj._sub._bacnet._tcp.", false);

try {
    DirectoryService.init();
    DirectoryService.getInstance().setDNSBinding(new DNSSD(ds));

} catch (final Exception e1) {
    // TODO Auto-generated catch block
    e1.printStackTrace();
}

final ReadPropertyRequest readRequest = new ReadPropertyRequest(
        new BACnetObjectIdentifier(BACnetObjectType.analogValue, 1),
        BACnetPropertyIdentifier.presentValue);
    

final ByteQueue byteQueue = new ByteQueue();
readRequest.write(byteQueue);
final TPDU tpdu = new TPDU(device1inStack1, device1inStack2,
        byteQueue.popAll());

final T_UnitDataRequest unitDataRequest = new T_UnitDataRequest(
        new URI("ws://localhost:9090"), tpdu, 1, true, null);

applicationService1.doRequest(unitDataRequest);
```


# Hands on 2

## Purpose
Hands on 2 demonstrates how to setup one BACnet/IT Stack on localhost with two BACnet devices. One BACnet device sends a WhoIsRequest to the other device. The WhoIsRequest is represented as a byte array, therefore no dependencies to other projects like BACnet4J (to represent BACnet service) is needed. The destinations address is already resolved (localhost), nevertheless the usage of a directory binding is shown.



## Download
1. Create a new empty directory __BACnetIT__ and make it the current directory
2. Download the source code of projects __ase__, __transport-binding-ws__ and __directory-binding-dnssd__.  
__ase__ project:  
```git clone https://github.com/fhnw-BACnet-IT/ase.git```  
__transport-binding-ws__ project:  
```git clone https://github.com/fhnw-BACnet-IT/transport-binding-ws.git```  
__directory-binding-dnssd__ project:  
```git clone https://github.com/fhnw-BACnet-IT/directory-binding-dnssd.git```  


## Build
1. Make __BACnetIT/transport-binding-ws__ the current directory.
2. Note that project __transport-binding-ws__ has a dependency to project __ase__, so ensure that both projects are stored at the same level in the __BACnetIT__ folder.
3. Build __transport-binding-ws__ using Gradle Wrapper:  
```
  ./gradlew clean build -x test
```  

4. Note that project __directory-binding-dnssd__ has a dependency to project __ase__ as well, so ensure that both projects are stored at the same level in the __BACnetIT__ folder.
5. Build __directory-binding-dnssd__ accordingly  
```
cd ../directory-binding-dnssd; ./gradlew build -x test;
```
6. Find all needed dependencies as jar files under __directory-binding-dnssd/build/distributions__ and
__transport-binding-ws/build/distributions__


## Setup
Setup a BACnet/IT Stack using Websocket as Transport Binding.  
This example doesn't use BACnet4j primitives, instead a WhoIsRequest is represented as a byte array.

### Preparation
Ensure the builded jars are in java class path.

#### Setup stack on localhost at port 8080
Consider [Hands On 1](#hands-on-1) to get more detailed information about the code below.

```java
final ConnectionFactory connectionFactory = new ConnectionFactory();

final int port = 8080;
connectionFactory.addConnectionClient("ws", new WSConnectionClientFactory());
connectionFactory.addConnectionServer("ws", new WSConnectionServerFactory(port));
    
final Channel channel1 = new Channel();
final ChannelConfiguration channelConfiguration1 = (ChannelConfiguration)channel1;
final ApplicationService applicationService1 = (ApplicationService)channel1;

final BACnetEID device1inStack1 = new BACnetEID(1001);
final BACnetEID device2inStack1 = new BACnetEID(1002);
final KeystoreConfig keystoreConfig1 = new KeystoreConfig("dummyKeystores/keyStoreDev1.jks","123456", "operationaldevcert");
final NetworkPortObj npo1 = new NetworkPortObj("ws", 8080, keystoreConfig1);

channelConfiguration1.registerChannelListener(new ChannelListener(device1inStack1) {
    @Override
    public void onIndication(
            final T_UnitDataIndication tUnitDataIndication,
            final ChannelHandlerContext ctx) {
        System.out.println(this.eid.getIdentifierAsString()
                + " got an indication" + tUnitDataIndication.getData());
    }

    @Override
    public void onError(final String cause) {
        System.err.println(cause);
    }

   
});

channelConfiguration1.registerChannelListener(new ChannelListener(device2inStack1) {
    @Override
    public void onIndication(
            final T_UnitDataIndication tUnitDataIndication,
            final ChannelHandlerContext ctx) {
        System.out.println(this.eid.getIdentifierAsString()
                + " got an indication" + tUnitDataIndication.getData());
    }

    @Override
    public void onError(final String cause) {
        System.err.println(cause);
    }

   
});

final BACnetEntityListener bacNetEntityHandler = new BACnetEntityListener() {

    @Override
    public void onRemoteAdded(final BACnetEID eid,
            final URI remoteUri) {
        DirectoryService.getInstance().register(eid, remoteUri, false,
                true);
    }

    @Override
    public void onRemoteRemove(final BACnetEID eid) {
        // TODO Auto-generated method stub
    }

    @Override
    public void onLocalRequested(final BACnetEID eid) {
        // TODO Auto-generated method stub
    }

};
channelConfiguration1.setEntityListener(bacNetEntityHandler);

channelConfiguration1.initializeAndStart(connectionFactory);

  

final DiscoveryConfig ds = new DiscoveryConfig(
        "DNSSD", "86.119.39.127",
        "itb.bacnet.ch.", "bds._sub._bacnet._tcp.",
        "dev._sub._bacnet._tcp.", "obj._sub._bacnet._tcp.", false);

try {
    DirectoryService.init();
    DirectoryService.getInstance().setDNSBinding(new DNSSD(ds));

} catch (final Exception e1) {
    // TODO Auto-generated catch block
    e1.printStackTrace();
}
    
   
    
// Get the byte stream of an WhoIsRequest()
byte[] whoIsRequest = new byte[]{(byte)0x1e,(byte)0x8e,(byte)0x8f,(byte)0x1f};
    
final TPDU tpdu = new TPDU(device1inStack1, device2inStack1, whoIsRequest);

final T_UnitDataRequest unitDataRequest = new T_UnitDataRequest(
        new URI("ws://localhost:8080"), tpdu, 1, true, null);

applicationService1.doRequest(unitDataRequest);
```


# Hands on 3
### Description / Story: 
Add two transport bindings; websocket and websocket secure.

```java
// Define key- and truststore
KeystoreConfig keystoreConfig = new KeystoreConfig([PATH-TO-KEYSTORE],
        [PWD], "operationaldevcert");
TruststoreConfig truststoreConfig = new TruststoreConfig([PATH-TO-TRUSTSTORE],
        [PWD], [TRUSTLIST]...);

// Build the connection factory
ConnectionFactory connectionFactory = new ConnectionFactory();

// Outgoing websocket secure transport binding (wss://)
connectionFactory.addConnectionClient("wss",
        new WSSConnectionClientFactory(keystoreConfig, truststoreConfig));
// Incoming websocket secure transport binding (wss://)
connectionFactory.addConnectionServer("wss",
        new WSSConnectionServerFactory([PORT], keystoreConfig, truststoreConfig));

// Outgoing websocket transport binding (ws://)    
connectionFactory.addConnectionClient("ws", new WSConnectionClientFactory());
// Incoming websocket transport binding (ws://)
connectionFactory.addConnectionServer("ws", new WSConnectionServerFactory([PORT]));
```

# Hands on 4
### Description / Story: 
Send a message between two simulated BACnet/IT devices using prior BACnet/IT EID resolution.

[follows]
        

