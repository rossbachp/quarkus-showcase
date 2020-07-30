# Quarkus Showcase

[![GitHub last commit](https://img.shields.io/github/last-commit/stephan-mueller/quarkus-showcase)](https://github.com/stephan-mueller/quarkus-showcase/commits) 
[![GitHub](https://img.shields.io/github/license/stephan-mueller/quarkus-showcase)](https://github.com/stephan-mueller/quarkus-showcase/blob/master/LICENSE)
[![CircleCI](https://circleci.com/gh/stephan-mueller/quarkus-showcase.svg?style=shield)](https://app.circleci.com/pipelines/github/stephan-mueller/quarkus-showcase)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=stephan-mueller_quarkus-showcase&metric=alert_status)](https://sonarcloud.io/dashboard?id=stephan-mueller_quarkus-showcase)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=stephan-mueller_quarkus-showcase&metric=coverage)](https://sonarcloud.io/dashboard?id=stephan-mueller_quarkus-showcase)

This is a showcase for the microservice framework [Quarkus](https://quarkus.io). It contains a hello world application, 
which demonstrates several features of Quarkus and Eclipse Microprofile.

Software requirements to run the samples are `maven`, `openjdk-8` (or any other JDK 8) and `docker`.
When running the Maven lifecycle it will create the war package and use the `quarkus-maven-plugin` to create a runnable 
JAR (fat JAR) which contains the application and the Quarkus application server. The fat JAR will be copied into a
Docker image using Spotify's `dockerfile-maven-plugin` during the package phase.

**Notable Features:**
* Dockerfiles for runnable JAR & Native Executable 
* Integration of MP Health, MP Metrics and MP OpenAPI
* Testcontainer tests with Rest-Assured, Cucumber and Postman/newman
* Code-Coverage for Testcontainer tests
* [CircleCI](https://circleci.com) Integration
* [Sonarcloud](https://sonarcloud.io) Integration


## How to run

Before running the application it needs to be compiled and packaged using `Maven`. It creates the runnable JAR and Docker image and can be 
run via `docker`:

```shell script
$ mvn clean package
$ docker run --rm -p 8080:8080 quarkus-showcase
```

If everything worked you can access the OpenAPI UI via [http://localhost:8080/swagger-ui](http://localhost:8080/swagger-ui).

### How to run a native image 

Before building a native executable and a native image, you have to install the GraalVM on your machine.

1. Install GraalVM (_Hint: check the Quarkus guide [building a native executable](https://quarkus.io/guides/building-native-image) for further details_)
2. Set GRAALVM_HOME environment variable to the GraalVM installation directory
3. Install the `native-image` tool using `gu install`

```shell script
$ export GRAALVM_HOME=/Library/Java/JavaVirtualMachines/graalvm-ce-java11-20.1.0/Contents/Home/
$ ${GRAALVM_HOME}/bin/gu install native-image
```

To create a docker image with a native executable you have to build the application with the Maven profile `native` 
```shell script
$ mvn clean package -Pnative
$ docker run --rm -p 8080:8080 quarkus-showcase
```

**Please note:**
 
The native executable is tailored for a specific operating system (Linux, macOS, Windows etc). 
If you build the native executable on macOS or Windows, you will not be able to use it inside a Linux based docker container. 

You can check the machine architecture with the following command:
```
$ file target/quarkus-showcase-runner
```
For a native executable build on macOs the output is:
> target/quarkus-showcase-runner: Mach-O 64-bit executable x86_64

For a native executable build in a Linux container the output is: 
> target/quarkus-showcase-runner: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/l, for GNU/Linux 3.2.0


To build a native executable which is usable inside a docker container you have to set the property 
`quarkus.native.container-build=true` (the property is set by default when using the Maven profile `native`). 
Instead of your machine a Linux container runtime is used for the build.    

If you want to try out the native executable on your machine use the following commands:
```shell script
$ mvn clean package -Pnative -Dquarkus.native.container-build=false -Ddockerfile.skip
$ ./target/quarkus-showcase-runner
```

### Resolving issues

Sometimes it may happen that the containers did not stop as expected when trying to stop the pipeline early. This may
result in running containers although they should have been stopped and removed. To detect them you need to check
Docker:

```shell script
$ docker ps -a | grep quarkus-showcase
```

If there are containers remaining although the application has been stopped you can remove them:

```shell script
$ docker rm <ids of the containers>
```


## Features

### Application 

The application is a very simple "Hello World" greeting service. It supports GET requests for generating a greeting message, and a PUT 
request for changing the greeting itself. The response is encoded using JSON.

Try the application
```shell script
curl -X GET http://localhost:8080/api/greet
{"message":"Hello World!"}

curl -X GET http://localhost:8080/api/greet/Stephan
{"message":"Hello Stephan!"}

curl -X PUT -H "Content-Type: application/json" -d '{"greeting" : "Hola"}' http://localhost:8080/api/greet/greeting

curl -X GET http://localhost:8080/api/greet/greeting
{"greeting":"Hola"}

curl -X GET http://localhost:8080/api/greet/Max
{"message":"Hola Max!"}
```

### Health, Metrics and OpenAPI

The application server provides built-in support for health, metrics and openapi endpoints.

Health liveness and readiness
```shell script
curl -s -X GET http://localhost:8080/health

curl -s -X GET http://localhost:8080/health/live

curl -s -X GET http://localhost:8080/health/ready
```

Metrics in Prometheus / JSON Format
```shell script
curl -s -X GET http://localhost:8080/metrics

curl -H 'Accept: application/json' -X GET http://localhost:8080/metrics
```

OpenAPI in YAML / JSON Format
```shell script
curl -s -X GET http://localhost:8080/openapi

curl -H 'Accept: application/json' -X GET http://localhost:8080/openapi
```