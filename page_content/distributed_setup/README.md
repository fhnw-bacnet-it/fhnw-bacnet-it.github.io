# Required changes for a distributed Setup
[Go back to start page](../../README.md)

To run this example in a distributed setup. (E.g. if you are using two computers and want them to communicate) note the following:

- __Changes in Configurator__:  
Adjust the destination ip address and port. Replace __localhost__ and __wsServerPort2__ with the corresponding information.  
Further check, that your device has EID 2001 and your communication partners device has EID 1001, otherwise adjust the EID in the code.


```java
// Represent a WhoIsRequest as byte array
byte[] whoIsRequest = new byte[]{(byte)0x1e,(byte)0x8e,(byte)0x8f,(byte)0x1f};
try{
    System.out.println("Applicatio2 sends a WhoIsRequest to Application1");
    application2.sendBACnetMessage(new URI("ws://[localhost]:"+[wsServerPort1]),
	     new BACnetEID([2001]), 
	     new BACnetEID([1001]),
	     whoIsRequest);
}catch(Exception e){
    System.out.println(e);
}
```

- __Changes in Application__:  
An application handles and answers to incoming messages. Therefore, change the URI to which the response shoud go.  
Replace __localhost__ and __port__ as well. Further check, that your device has EID 1001 and your communication partners device has EID 2001, otherwise adjust the EID in the code.

```java
// Dummy Handling of a WhoIsRequest
else if (incoming instanceof UnconfirmedRequest 
	&& ((UnconfirmedRequest) incoming).getService() instanceof WhoIsRequest) {
    System.out.println("Application1 got an indication - WhoIsRequest");

    // Represent an IAmRequest as byte array
    byte[] iAmRequest = new byte[] { (byte) 0x1E, (byte) 0x0E,
            (byte) 0xC4, (byte) 0x02, (byte) 0x00, (byte) 0x00,
            (byte) 0x00, (byte) 0x21, (byte) 0x01, (byte) 0x91,
            (byte) 0x00, (byte) 0x21, (byte) 0x01, (byte) 0x0F,
            (byte) 0x1F };

    try {
        System.out.println("Application1 sends an IAmRequest to Application2");
        sendBACnetMessage(new URI("ws://[localhost]:[9090]"),
                new BACnetEID([1001]), new BACnetEID([2001]),iAmRequest);
    } catch (Exception e) {
    }

}
```

