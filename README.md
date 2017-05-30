# About BACnet/IT

BACnet/IT has been developed at the [Institute of Mobile and Distributed Systems of the University of Applied Sciences Northwestern Switzerland](http://www.fhnw.ch/engineering/imvs).

# BACnet/IT Tutorial

__In this tutorial you will learn how to develop applications using our new BACnet/IT stack.__

## Content
[What we do](#what-we-do)  
[Download Binaries and Prerequisites](page_content/download/README.md)  
[Create a Demo Applicaton without Directory Service](page_content/demo_app_no_directory/README.md)  
[Create a Demo Application using the Directory Service](page_content/demo_app_directory/README.md)  
[Build from source](page_content/build/README.md)  
[Required changes for a distributed setup](page_content/distributed_setup/README.md)


## What we do

Consider the following figure:  

<img src="page_content/images/overview.png" width="80%">  


We will create two applications with each two simulated BACnet devices (yellow and green).  
Both applications will extend the abstract class __AbstractApplication__, which is not shown in the figure,  
__AbstractApplication__ provides base functionality to the applications.  
Each application uses an __Application Service Element (ASE)__ to communicate with other applications.  
The __ASE__ component is given but has to be configured. Therefore, we will create the class __Configurator__ (blue).  

The __configurator__ has four fundamental tasks:  

1. Initialize the necessary BACnet/IT ASE components.  
2. Link the two applications with the corresponding communication stacks. (in figure: ASEs, white).  
3. Link the available Transport Bindings (gray) to the ASEs.
4. Enforce communication between devices.

As mentioned in point 4, after the setup of the classes, we let communicate device 1001 (yellow) with device 2001 (green) and vice versa.


Note the two interfaces __ApplicationService__ and __ChannelConfiguration__ and remember the two basic rules:  
1. The Applications's view of ASE is ApplicationService.  
2. The Configurator's view of ASE is ChannelConfiguration.

And the last point before we start:  
All the classes you need to develop any applications, as well as this demo setup, are packed into "api" packages.  
Thus, ensure that all your imported classes that consider components of the BACnet/IT libraries are within an api package.

__With this information in mind, let us start and build our first BACnet/IT application!__

[Go to Download Binaries and Prerequisites](page_content/download/README.md)  