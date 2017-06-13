# Build from source
[Go back to start page](../../README.md)

Instead of downloading the binaries you can build the JAR files from source.  
We are working with [Gradle Build Tool](https://gradle.org).  
For simplicity we provide Gradle Wrappers, therefore you don't have to install Gradle Build Tool.  
Ensure your JDK is set and follow the steps to build the projects.


## Get the source

1. Create a new empty directory __BACnetIT__ and make it the current directory.
2. Clone the following GitHub projects:  
__ase__ project:  
```https://github.com/fhnw-bacnet-it/ase.git```  
__transport-binding-ws__ project:  
```https://github.com/fhnw-bacnet-it/transport-binding-ws.git```  
__directory-binding-dnssd__ project:  
```https://github.com/fhnw-bacnet-it/directory-binding-dnssd.git```  
__samples-and-tests__ project:  
```https://github.com/fhnw-bacnet-it/samples-and-tests.git```


Alternatively use the following commands to checkout all the projects.  
__MAC OSX or LINUX:__

```shell
mkdir -p ~/BACnetIT/
cd ~/BACnetIT/
git clone https://github.com/fhnw-bacnet-it/ase.git
git clone https://github.com/fhnw-bacnet-it/transport-binding-ws.git
git clone https://github.com/fhnw-BACnet-IT/directory-binding-dnssd.git
git clone https://github.com/fhnw-BACnet-IT/samples-and-tests.git
```
__WINDOWS:__

```shell
mkdir %HOMEPATH%\BACnetIT\
cd %HOMEPATH%\BACnetIT\
git clone https://github.com/fhnw-bacnet-it/ase.git
git clone https://github.com/fhnw-bacnet-it/transport-binding-ws.git
git clone https://github.com/fhnw-BACnet-IT/directory-binding-dnssd.git
git clone https://github.com/fhnw-BACnet-IT/samples-and-tests.git
```


## Build the Source using Gradle Wrapper

1. Make __BACnetIT/samples-and-tests__ the current directory.
2. Note that project __samples-and-tests__ depends on the projects __ase__, __transport-binding-ws__ and __directory-binding-dnssd__, thus ensure that all projects are stored at the same level in the __BACnetIT__ folder.
3. Build __samples-and-tests__ using Gradle Wrapper:  

__MAC OSX or LINUX:__

```shell
  cd samples-and-tests
  ./gradlew clean build -x test
```

__WINDOWS:__


```shell
  cd samples-and-tests
  gradlew.bat clean build -x test
```

4. Find all needed dependencies as JAR files in a ZIP archive under __samples-and-tests/build/distributions__.

[Go to Required changes for a distributed setup](../6_distributed_setup/README.md)