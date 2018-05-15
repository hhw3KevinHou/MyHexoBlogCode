---
title: Spring Cloud
tags: Spring Cloud
categories: Spring
---

## bootstrap.yml(bootstrap.properties) & application.yml(application.properties)

A Spring Cloud application operates by creating a "bootstrap" context, which is a parent context for the main application. Out of the box it is responsible for loading configuration properties from the external sources, and also decrypting properties in the local external configuration files.
##### Typically bootstrap.yml(bootstrap.properties) contains two properties:

* location of the configuration server (spring.cloud.config.uri)
* name of the application (spring.application.name)

Upon startup, Spring Cloud makes an HTTP call to the config server with the name of the application and retrieves back that application's configuration.

##### application.yml(application.properties)
Contains standard application configuration - typically default configuration since any configuration retrieved during the bootstrap process will override configuration defined here.

bootstrap.yml (or properties) have lower precedence than the application.yml (or properties) and any other property sources that are added to the child as a normal part of the process of creating a Spring Boot application. 

#Spring Cloud Context 
provides utilities and special services for the ApplicationContext of a Spring Cloud application (bootstrap context, encryption, refresh scope and environment endpoints). 

Normal Spring application context behaviour rules apply to property resolution: properties from a child context override those in the parent, by name and also by property source name (if the child has a property source with the same name as the parent, the one from the parent is not included in the child).

#Endpoints

For a Spring Boot Actuator application there are some additional management endpoints:

POST to /env to update the Environment and rebind @ConfigurationProperties and log levels
/refresh for re-loading the boot strap context and refreshing the @RefreshScope beans
/restart for closing the ApplicationContext and restarting it (disabled by default)
/pause and /resume for calling the Lifecycle methods (stop() and start() on the ApplicationContext)

#Spring Cloud Commons 
is a set of abstractions and common classes used in different Spring Cloud implementations (eg. Spring Cloud Netflix vs. Spring Cloud Consul).



