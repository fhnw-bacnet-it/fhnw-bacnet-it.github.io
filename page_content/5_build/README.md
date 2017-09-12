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

Because of project dependencies the projects need to be built in a specific order.
As of release 0.10 BACnet-IT dependencies are no longer referenced as included Gradle projects,
but instead as JARs from Maven repositories. When building from source, the compiled JARs must be
published to the local Maven repository using the `publishToMavenLocal` task.

The __BACnet-IT__ projects should be built in the following order:

1. __ase__
2. __transport-binding-ws__
3. __directory-binding-dnssd__
4. __samples-and-tests__

Use the Gradle wrapper as shown below: 

__MAC OSX or LINUX:__

```shell
  cd ~/BACnetIT
  cd ./ase
  ./gradlew clean build publishToMavenLocal
  cd ../transport-binding-ws
  ./gradlew clean build publishToMavenLocal
  cd ../directory-binding-dnssd
  ./gradlew clean build publishToMavenLocal
  cd ../samples-and-tests
  ./gradlew clean build
```

__WINDOWS:__

```shell
  cd %HOMEPATH%\BACnetIT
  cd .\ase
  gradlew.bat clean build publishToMavenLocal
  cd ..\transport-binding-ws
  gradlew.bat clean build publishToMavenLocal
  cd ..\directory-binding-dnssd
  gradlew.bat clean build publishToMavenLocal
  cd ..\samples-and-tests
  gradlew.bat clean build
```
The __samples-and-tests__ project builds a ZIP archive containing all required JARs under __samples-and-tests/build/distributions__.

## Build from Source using Jenkins CI

If you want to use a continuous integration solution, such as Jenkins, an example is provided below.
The given example provides a pipeline script that can be copied into a Jenkins pipeline job.

```groovy
pipeline {

    // Make sure the required tools are available
    tools {
        jdk "Oracle JDK 8u121"
    }
    
    // environment variables
    environment {
        GRADLE_OPTS = '-Dorg.gradle.daemon=false'
        GRADLE_PROPS = ''
    }
    
    // Run on any executor
    agent any 

    // The pipeline stages
    stages {
        stage('Preparation') {
            steps {
                dir('ase') {
                    git url: 'https://github.com/fhnw-bacnet-it/ase.git', branch: 'master'
                }
                dir('transport-binding-ws') {
                    git url: 'https://github.com/fhnw-bacnet-it/transport-binding-ws.git', branch: 'master'
                }
                dir('directory-binding-dnssd') {
                    git url: 'https://github.com/fhnw-bacnet-it/directory-binding-dnssd.git', branch: 'master'
                }
                dir('samples-and-tests') {
                    git url: 'https://github.com/fhnw-bacnet-it/samples-and-tests.git', branch: 'master'
                }
            }
        }
        stage('Build') {
            steps {
                dir('ase') {
                    sh "./gradlew clean build publishToMavenLocal"
                }
                dir('transport-binding-ws') {
                    sh "./gradlew clean build publishToMavenLocal"
                }
                dir('directory-binding-dnssd') {
                    sh "./gradlew clean build publishToMavenLocal"
                }
                dir('samples-and-tests') {
                    sh "./gradlew clean build"
                }
            }
        }
    }
    
    // Is executed after the build process
    post {
        always {
            // example: publish test results
            dir('samples-and-tests') {
                junit 'build/test-results/test/*.xml'
            }
        }
    }
}
```

## Distributed Environment

[Go to Required changes for a distributed setup](../6_distributed_setup/README.md)
