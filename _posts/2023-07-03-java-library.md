---
layout: post
title: Create a Java library
date: 2023-07-03 21:21:20 +0900
# description: 
# img: 
# fig-caption: 
tags: [java, maven]
---

## Install JDK

- Download JDK from [here](https://www.oracle.com/java/technologies/downloads/#jdk20-mac)
- If JDK needs to be [uninstalled](https://docs.oracle.com/en/java/javase/20/install/installation-jdk-macos.html#GUID-F9183C70-2E96-40F4-9104-F3814A5A331F)
- If gradle is used as the build tool, JDK 20 is not supported yet (2023 July)
- Check after installation: ```java -version```

## Install Maven

## Create a Maven library
- follow this [tutorial](https://gemfury.com/guide/maven/)
- Maven commands
    - create the project: ```mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false```
    - install the library in local repository: ```mvn clean install```
    - run the app: ```mvn pacakge; java -cp target/examples-0.1.jar com.chiral.examples.App```
 

## gRPC for Java
- [Quickstart from grpc-java](https://grpc.io/docs/languages/java/quickstart/)
- To install the plugin of protoc-gen-grpc-java, download from [here](https://repo1.maven.org/maven2/io/grpc/protoc-gen-grpc-java)