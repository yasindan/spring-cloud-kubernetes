## Spring Cloud Kubernetes

Spring Cloud integration with Kubernetes

### Features

-   [DiscoveryClient for Kubernetes](#discoveryclient-for-kubernetes)
-   [KubernetesClient autoconfiguration](#kubernetesclient-autoconfiguration)
-   [ConfigMap PropertySource](#configmap-propertysource)
-   [Pod Health Indicator](#pod-health-indicator)
-   [Transparency](#transparency) *(its transparent wether the code runs in or outside of Kubernetes)* 
-   [Kubernetes Profile Autoconfiguration](#kubernetes-profile-autoconfiguration)
-   [Ribbon discovery in Kubernetes](#ribbon-discovery-in-kubernetes)
-   [Zipkin discovery in Kubernetes](#zipkin-discovery-in-kubernetes)
-   [ConfigMap Archaius Bridge](#configmap-archaius-bridge)

---
### DiscoveryClient for Kubernetes

[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.fabric8/spring-cloud-starter-kubernetes/badge.svg?style=flat-square)](https://maven-badges.herokuapp.com/maven-central/io.fabric8/spring-cloud-starter-kubernetes/)
[![Javadocs](http://www.javadoc.io/badge/io.fabric8/spring-cloud-starter-kubernetes.svg?color=blue)](http://www.javadoc.io/doc/io.fabric8/spring-cloud-starter-kubernetes)
[![Dependency Status](https://www.versioneye.com/java/io.fabric8:spring-cloud-starter-kubernetes/badge?style=flat)](https://www.versioneye.com/java/io.fabric8:spring-cloud-starter-kubernetes/)


This project provides an implementation of [Discovery Client](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-commons/src/main/java/org/springframework/cloud/client/discovery/DiscoveryClient.java) for [Kubernetes](http://kubernetes.io). This allows you to query Kubernetes endpoints *(see [services](http://kubernetes.io/docs/user-guide/services/))* by name. 
This is something that you get for free just by adding the following dependency inside your project:

```xml
<dependency>
    <groupId>io.fabric8</groupId>
    <artifactId>spring-cloud-starter-kubernetes</artifactId>
    <version>${latest.version></version>
</dependency>
```

Then you can inject the client in your cloud simply by:

```java
@Autowire
private DiscoveryClient discoveryClient;
```

If for any reason you need to disable the `DiscoveryClient` you can simply set the following property:

```
spring.cloud.kubernetes.discovery.enabled=false
```

Some spring cloud components use the `DiscoveryClient` in order obtain info about the local service instance. For this to work you need to align the service name with `spring.application.name`.

#### ConfigMap PropertySource

The most common approach to configure your spring boot application is to edit the `application.yaml` file. Often the user may override properties by specifying system properties or env variables.
Kubernetes has the notion of [ConfigMap](http://kubernetes.io/docs/user-guide/configmap/) for passing configuration to the application. This project provides integration with `ConfigMap` to make config maps accessible by spring boot.

The `ConfigMap` `PropertySource` when enabled will lookup Kubernetes for a `ConfigMap` named after the application (see `spring.application.name`). If the map is found it will read its data and do the following:

- apply individual configuration properties.
- apply as yaml the content of any property named `application.yaml`
- apple as properties file the content of any property named `application.properties`

Example:

Let's assume that we have a spring boot application named ``demo`` that uses properties to read its thread pool configuration.

- `pool.size.core`
- `pool.size.maximum`

This can be externalized to config map in yaml format:

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: demo
data:
  pool.size.core: 1
  pool.size.max: 16
```    

Individual properties work fine for most cases but sometimes we yaml is more convinient. In this case we will use a single property named `application.yaml` and embed our yaml inside it:
 
 ```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: demo
data:
  application.yaml: |-
    pool:
      size:
        core: 1
        max:16
``` 

#### Pod Health Indicator

Spring Boot uses [HealthIndicator](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java) to expose info about the health of an application.
That makes it really useful for exposing health related information to the user and are also a good fit for use as [readiness probes](http://kubernetes.io/docs/user-guide/production-pods/#liveness-and-readiness-probes-aka-health-checks).

The Kubernetes health indicator which is part of the core modules exposes the following info:

- pod name
- visible services
- flag that indicates if app is internal or external to Kubernetes

#### Transparency 

All of the features described above will work equally fine regardless of wether our application is inside Kubernetes or not. This is really helpful for development and troubleshooting.

#### Kubernetes Profile Autoconfiguration

When the application is run inside Kubernetes a profile named `kubernetes` will automatically get activated. 
This allows the user to customize the configuration that will be applied in and out of kubernetes *(e.g. different dev and prod configuration)*.

### Ribbon discovery in Kubernetes

[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.fabric8/spring-cloud-starter-kubernetes-netflix/badge.svg?style=flat-square)](https://maven-badges.herokuapp.com/maven-central/io.fabric8/spring-cloud-starter-kubernetes-netflix/)
[![Javadocs](http://www.javadoc.io/badge/io.fabric8/spring-cloud-starter-kubernetes-netflix.svg?color=blue)](http://www.javadoc.io/doc/io.fabric8/spring-cloud-starter-kubernetes-netflix)
[![Dependency Status](https://www.versioneye.com/java/io.fabric8:spring-cloud-starter-kubernetes-netflix/badge?style=flat)](https://www.versioneye.com/java/io.fabric8:spring-cloud-starter-kubernetes-netflix/)

A Kubernetes based `ServerList` for Ribbon has been implemented. The implementation is part of the [spring-cloud-kubernetes-ribbon](spring-cloud-kubernetes-ribbon/pom.xml) module and you can use it by adding:

```xml
<dependency>
    <groupId>io.fabric8</groupId>
    <artifactId>spring-cloud-starter-kubernetes-netflix</artifactId>
    <version>${latest.version></version>
</dependency>
```

The ribbon discovery client can be disabled by setting `spring.cloud.kubernetes.ribbon.enabled=false`.

By default the client will detect all endpoints with the configured *client name* that lives in the current namespace.
If the endpoint contains multiple ports, the first port will be used. To fine tune the name of the desired port (if the service is a multiport service) or fine tune the namespace you can use one of the following properties.

- `KubernetesNamespace`
- `PortName`

Examples that are using this module for ribbon discovery are:

- [iPaas Quickstarts - Spring Boot - Ribbon](https://github.com/fabric8io/ipaas-quickstarts/tree/master/quickstart/spring-boot/ribbon)
- [Kubeflix - LoanBroker - Bank](https://github.com/fabric8io/kubeflix/tree/master/examples/loanbroker/bank) 


### Zipkin discovery in Kubernetes

[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.fabric8/spring-cloud-starter-kubernetes-zipkin/badge.svg?style=flat-square)](https://maven-badges.herokuapp.com/maven-central/io.fabric8/spring-cloud-starter-kubernetes-zipkin/)
[![Javadocs](http://www.javadoc.io/badge/io.fabric8/spring-cloud-starter-kubernetes-zipkin.svg?color=blue)](http://www.javadoc.io/doc/io.fabric8/spring-cloud-starter-kubernetes-zipkin)
[![Dependency Status](https://www.versioneye.com/java/io.fabric8:spring-cloud-starter-kubernetes-zipkin/badge?style=flat)](https://www.versioneye.com/java/io.fabric8:spring-cloud-starter-kubernetes-zipkin/)

[Zipkin](https://github.com/openzipkin/zipkin) is a distributed tracing system and it is also supported by [Sleuth](https://github.com/spring-cloud/spring-cloud-sleuth).

Discovery of the services required by Zipkin (e.g. `zipkin-query`) is provided by [spring-cloud-kubernetes-zipkin](spring-cloud-kubernetes-zipkin/pom.xml) module and you can use it by adding:

```xml
<dependency>
    <groupId>io.fabric8</groupId>
    <artifactId>spring-cloud-starter-kubernetes-zipkin</artifactId>
    <version>${latest.version></version>
</dependency>
```

This works as an extension of [spring-cloud-sleuth-zipkin](https://github.com/spring-cloud/spring-cloud-sleuth/tree/master/spring-cloud-sleuth-zipkin).    

Examples of application that are using Zipkin discovery in Kubernetes:

- [iPaas Quickstarts - Spring Boot - Ribbon](https://github.com/fabric8io/ipaas-quickstarts/tree/master/quickstart/spring-boot/ribbon)
- [Kubeflix - LoanBroker - Bank](https://github.com/fabric8io/kubeflix/tree/master/examples/loanbroker/bank) 

### ConfigMap Archaius Bridge

[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.fabric8/spring-cloud-kubernetes-archaius/badge.svg?style=flat-square)](https://maven-badges.herokuapp.com/maven-central/io.fabric8/spring-cloud-kubernetes-archaius/)
[![Javadocs](http://www.javadoc.io/badge/io.fabric8/spring-cloud-kubernetes-archaius.svg?color=blue)](http://www.javadoc.io/doc/io.fabric8/spring-cloud-kubernetes-archaius)
[![Dependency Status](https://www.versioneye.com/java/io.fabric8:spring-cloud-kubernetes-archaius/badge?style=flat)](https://www.versioneye.com/java/io.fabric8:spring-cloud-kubernetes-archaius/)

Section [ConfigMap PropertySource](#configmap-propertysource) provides a brief explanation on how to configure spring boot application via ConfigMap.
This approach will aid in creating the configuration properties objects that will be passed in our application. If our application is using Archaius it will indirectly benefit by it.
An alternative approach that provides more direct Archaius support without getting in the way of spring configuration properties by using [spring-cloud-kubernetes-archaius](spring-cloud-kubernetes-archaius/pom.xml) that is part of the Netflix starter. 

This module allows you to annotate your application with the `@ArchaiusConfigMapSource` and archaius will automatically use the configmap as a watched source *(get notification on changes)*.

---
### Building

You can just use maven to build it from sources:

```
mvn clean install
```    
    
### Usage

The project provides a "starter" module, so you just need to add the following dependency in your project.

```xml
<dependency>
    <groupId>io.fabric8</groupId>
    <artifactId>spring-cloud-starter-kubernetes</artifactId>
    <version>x.y.z</version>
</dependency>
```    
