# Fallwizard

[![Build Status](https://travis-ci.org/Fallwizard/Fallwizard.svg)](https://travis-ci.org/Fallwizard/Fallwizard)

I like candy!!!

Leveraging the best of the Spring Framework and Dropwizard.

__Maven Dependency__
```xml
<dependencies>
  <dependency>
    <groupId>io.github.fallwizard</groupId>
    <artifactId>Fallwizard</artifactId>
    <version>0.7.1.2</version>
  </dependency>
</dependencies>
```

## Why?

We love Spring Framework because it simplified Enterprise Java.  We love Dropwizard because it simplified App Servers.  What if these two frameworks made sweet-sweet love and had a child?  Well, you would get Fallwizard.

Ok seriously...

Fallwizard is the integration of Spring with Dropwizard.  This gives us the following benefits we were missing when using pure Dropwizard:

* Ability to support complex configurations outside of the yaml mechanism (i.e. we need more dynamicism).
* Better support for complex authentication and authorization environments.

> This is not a condemnation of Dropwizard.  Most organizations don't need the flexibility we need in configuration and security.

So, what's up with the name?

Spring -> Fall <- Drop...wizard.  Get it?  (Ok, it's corny.)

## Why not Spring Boot?

Right up front, let us say that we love Spring Boot.  Remember, though, that Fallwizard (and Dropwizard) predate Spring Boot.

So not only do we wish to keep supporting Fallwizard, we find that some things are still easier with dropwizard than Spring Boot:
* Packaging into RPMs / DEBs
* Ops: Healthchecks, Metrics, etc.

So it really just depends on your own preferences and the legacy of your already-in-production projects.

## Features

* Define Resources, Healthchecks, Tasks, etc. in Spring, and have them automatically bootstrapped in Dropwizard.
* Utilize Spring Security (out of the box) and have it applied to Dropwizard routes.
* Automatically inject a Spring Security UserDetails object into REST methods using Dropwizard's `@Auth` annotation.

## Changelog

#### Release 2.0.2

* The first release to be uploaded to maven central
* **NOTE:** All namespaces have been changed from com.bericotech to io.github.fallwizard

#### Release 2.0.1

* Updates the generated pom in preparation for upload to maven central

#### Release 2.0.0

* Updated to the lastest dropwizard (0.7.1)!
* Updated spring dependencies to use [Spring IO Platform](http://platform.spring.io/platform/) version 1.0.1.RELEASE
* Updated to use gradle 2.0

#### Release 1.3.0

* Adds properties files into the spring context so that you can use PropertyPlaceholderConfigurer in your contexts
* Fixed a bug that was keeping certain spring security configurations from having a servlet filter applied
 
#### Release 1.2.0

* Fixed a bug with the bean profiles
* Changed berico package-name and artifact group to use bericotech  (e.g. maven group is now com.bericotech)
* Restructured the configuration file so that the spring configuration elements follow the DropWizard pattern and are organized in their own section.  See the example YAML configuration file below.
* Added the ability to inject properties/specify property location files into Spring context directly from the YAML config file.

#### Release 1.1.0

* Added Support for Bean Profiles.  Now you can specify a "beanProfiles" property (Array of String) in the Yaml config to specify the Bean Profiles you want to use.  This should eliminate the need of maintaining different lists of context files for production/dev/integation configurations.
* No more reliance on `FileSystemXmlApplicationContext`.  We have switched to favor `GenericXmlApplicationContext`, which will allow you to use `file`, `classpath`, and `url` resources.  __I made one opinionated change to the way Spring handle configuration by default!__  If an application context file is not "prefixed" with either `file`, `classpath`, or `url`, Fallwizard will default to `file` NOT `classpath`.  The assumption is that you will probably have your configuration in a subdirectory outside of the FAT JAR.

#### Release 1.0.0

* Spring contexts loaded from yaml specified xml files.
* Spring Security implementation of the Dropwizard Authentication plugin.

## Related

__One of our engineers, Travis, has developed a Gradle Plugin for building Dropwizard projects__

https://github.com/Fallwizard/dropwizard-gradle-plugin


## Using Fallwizard

### Configuring Fallwizard

#### Fallwizard Conventions

Instead of using Dropwizard's `Application` and `Configuration` classes, you should extend (if you want) `FallwizardApplication` and `FallwizardConfiguration`.  If you set your configuration values and wireup your resources in Spring, you actually don't need to extend `FallwizardApplication` (just specify this as your main class in Maven/Gradle).

If you need to add Dropwizard bundles to your service, you will need to extend `FallwizardApplication`; please remember to call `super` on any method you override.

#### Yaml Configuration

```yaml

# Spring Specific Configuration
spring:
    # Application Contexts to Load.
    applicationContext: ['conf/applicationContext.xml', 'file:conf/basicAuthSecurityContext.xml',
    'classpath:sprocketContext.xml', 'url:http://conf.berico.us/sproket-manager/context.xml']

    # [Optional] Which bean profiles to use?
    #  Bean Profiles allow you to alter the beans which the application
    #  context files uses/instantiates based on your arranged profile.
    #  Think "dev", "test", "production" as typical system profiles
    #  and/or choosing between bing/google/other configurations
    #  as example feature profiles.
    #
    #  See http://spring.io/blog/2011/02/11/spring-framework-3-1-m1-released/  as a good referene.
    #
    beanProfiles: ['production', 'feature1', 'feature2']

    # Should Spring Security be used?
    useSpringSecurity: true

    # [Optional] Allow property injection based on this YAML file.
    # Sometimes you wish the YAML configuration to affect the configuration of the
    # Spring file, and this allows you to set the property values directly in this
    # YAML file, or which property files should be used from the YAML file.
    #
    # See (for more info about behavior):
    # http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/config/PropertyPlaceholderConfigurer.html
    #
    propertyPlaceholderConfigurer:

         # [Optional] Defaults to: false
         # use this if you wish to ignore properties that don't resolve
         # (E.g. ${property.notfound}  will not cause an error.
         # This is useful when there are other PropertyPlaceholderConfigurers
         # in the Spring Files.
         ignoreUnresolvablePlaceholders: true

         # [Optional] Again useful with other PropertyPlaecholder Configurers, this
         # sets the ordering of resolution (to determine which Configurer
         # overwrites values set by previous configurers.)  Only useful if you
         # know the order of embedded PropertyPlaceHolder configurers.
         order: 123

         # [Optional]
         # Set properties directly, that will then be available in the
         # application context files in bean parameters like
         #    value="${encrypted.passwd}"
         properties:
             "property.key": "my magic key"
             "property.value": "my magic box"
             "encrytped.passwd" : "<my encrypted password>"
             "some.other.property" : "with its magic value"
             "etc" : "etc value!"

         # [Optional]
         # You can specify a LIST of property files (which in turn just have straight properties).
         locations:
             - file:/my/home/path/config/production.properties
             - file:/my/home/path/config/googleMaps.properties

# This might be a custom property of yours if you extended SpringConfiguration.
exampleProperty: This was your example property


########## Everything else is basic Dropwizard ###################

logging:
    level: INFO
    console:
        enabled: true
        threshold: DEBUG

http:
    port: 8080
    adminUsername: admin
    adminPassword: admin
    rootPath: /service/*
```

#### Spring Configuration

Wire up all of your JAX-RS Resources, Dropwizard Tasks, Health Checks, Managed Services, etc. in Spring.  Fallwizard will look for classes implementing Dropwizard interfaces or classes and automatically pull them out of the Spring Context and register them with Dropwizard.

### Spring Security Integration

We needed certificated-based client authentication, but didn't want to force BasicAuth when we were already using SSL just so we could inject an object (i.e. `@Auth`) in our JAX-RS controllers.

We also wanted the rich features of Spring Security, which includes support for numerous authentication mechanisms.

__Fallwizard will automatically find the necessary Spring Security components and bind them to Dropwizard if `useSpringSecurity: true` is specified in the yaml configuration__

You simply need to have a Spring Security configuration that looks something like this in your applicationContext.xml:

_Example for Certificate-based authentication_

```xml
<security:http>
  <security:intercept-url pattern="/*" access="ROLE_USER" />
  <security:intercept-url pattern="/admin/*" access="ROLE_ADMIN" />
  <security:x509 subject-principal-regex="CN=(.*?)," />
</security:http>

<security:authentication-manager>
  <security:authentication-provider>
    <security:user-service>
      <security:user name="Super Awesome Client" authorities="ROLE_USER" />
      <security:user name="The Boss" authorities="ROLE_USER, ROLE_ADMIN" />
    </security:user-service>
  </security:authentication-provider>
</security:authentication-manager>
```

Once you've got Spring Security working, you can use `@Auth UserDetails userDetails` in your JAX-RS controllers.

Here's an examply of injecting the Spring Security `UserDetails` context into one your controllers (why invent a new `User` object?).

```java
@GET
@Timed
@Path("/chitty-chat/{topic}")
public ChittyChat getChittyChatOnTopic(@Auth UserDetails userDetails, @PathParam("topic") String topic){
   // ... get Chitty-Chat ...
}
```

That's it!

## Using Spring Security without Fallwizard

If you don't like our stuff and prefer to just using Spring Security, you will need to register the `SpringSecurityAuthProvider` with Dropwizard manually.  It looks something like this:

#### Initialize your Spring Application Context.

Initialize your Spring Application Context in your Dropwizard `Application` class.  We explicitly require the location of the `applicationContext` in our Dropwizard `Configuration` class.

```java
SpringConfiguration springConfig = configuration.getSpringConfiguration();
ApplicationContext applicationContext =
  new FileSystemXmlApplicationContext(
    springConfig.getSpringApplicationContext());
```

#### Register Spring Security with the Dropwizard `Environment`.

```java
@Override
public void run(BlahBlahConfiguration configuration, Environment environment) throws Exception {

  SpringConfiguration springConfig = configuration.getSpringConfiguration();
  ApplicationContext applicationContext =
    new FileSystemXmlApplicationContext(
      springConfig.getSpringApplicationContext());

  new SpringSecurityAuthProvider(applicationContext).registerProvider(environment);
}
```



## Enjoy a cold drink, you deserve it.

You're done.  Now you can muck with the Spring ApplicationContext "outside" of Dropwizard.
