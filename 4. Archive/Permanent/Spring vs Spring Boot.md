---
created: 2023-08-03 17:16
tags: java, frameworks
---
# Spring vs Spring Boot

## What's Spring?

Framework, provides comprehensive *infrastructure support* to Java applications; packed with features like DI, JDBC, MVC, Security, [[Spring Test]] etc. to reduce dev time of an application.

## What's [[Spring Boot]]?

An extension of the Spring framework, with an **opinionated view** of the platform.

Features:
+ "Starter" dependencies (opinionated)
+ Embedded server
+ Metrics, health check
+ Externalized configuration
	+ https://docs.spring.io/spring-boot/docs/2.1.13.RELEASE/reference/html/boot-features-external-config.html
+ Automatic config for Spring functionality wherever possible.

## Dependencies

Spring: `spring-web` + `spring-webmvc` at minimum.

Spring Boot: Just `spring-boot-starter-web`. This brings in Spring Test, JUnit, Mockito, Hamcrest etc. for testing, along with other dependencies.

Full `spring-boot-starter-web` deptree:

```
\- org.springframework.boot:spring-boot-starter-web:jar:3.1.2:compile
   +- org.springframework.boot:spring-boot-starter:jar:3.1.2:compile
   |  +- org.springframework.boot:spring-boot:jar:3.1.2:compile
   |  +- org.springframework.boot:spring-boot-autoconfigure:jar:3.1.2:compile
   |  +- org.springframework.boot:spring-boot-starter-logging:jar:3.1.2:compile
   |  |  +- ch.qos.logback:logback-classic:jar:1.4.8:compile
   |  |  |  +- ch.qos.logback:logback-core:jar:1.4.8:compile
   |  |  |  \- org.slf4j:slf4j-api:jar:2.0.7:compile
   |  |  +- org.apache.logging.log4j:log4j-to-slf4j:jar:2.20.0:compile
   |  |  |  \- org.apache.logging.log4j:log4j-api:jar:2.20.0:compile
   |  |  \- org.slf4j:jul-to-slf4j:jar:2.0.7:compile
   |  +- jakarta.annotation:jakarta.annotation-api:jar:2.1.1:compile
   |  +- org.springframework:spring-core:jar:6.0.11:compile
   |  |  \- org.springframework:spring-jcl:jar:6.0.11:compile
   |  \- org.yaml:snakeyaml:jar:1.33:compile
   +- org.springframework.boot:spring-boot-starter-json:jar:3.1.2:compile
   |  +- com.fasterxml.jackson.core:jackson-databind:jar:2.15.2:compile
   |  |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.15.2:compile
   |  |  \- com.fasterxml.jackson.core:jackson-core:jar:2.15.2:compile
   |  +- com.fasterxml.jackson.datatype:jackson-datatype-jdk8:jar:2.15.2:compile
   |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.15.2:compile
   |  \- com.fasterxml.jackson.module:jackson-module-parameter-names:jar:2.15.2:compile
   +- org.springframework.boot:spring-boot-starter-tomcat:jar:3.1.2:compile
   |  +- org.apache.tomcat.embed:tomcat-embed-core:jar:10.1.11:compile
   |  +- org.apache.tomcat.embed:tomcat-embed-el:jar:10.1.11:compile
   |  \- org.apache.tomcat.embed:tomcat-embed-websocket:jar:10.1.11:compile
   +- org.springframework:spring-web:jar:6.0.11:compile
   |  +- org.springframework:spring-beans:jar:6.0.11:compile
   |  \- io.micrometer:micrometer-observation:jar:1.10.9:compile
   |     \- io.micrometer:micrometer-commons:jar:1.10.9:compile
   \- org.springframework:spring-webmvc:jar:6.0.11:compile
      +- org.springframework:spring-aop:jar:6.0.11:compile
      +- org.springframework:spring-context:jar:6.0.11:compile
      \- org.springframework:spring-expression:jar:6.0.11:compile
```

## MVC Configuration

**Spring requires defining the dispatcher servlet, mappings, and other supporting configurations**. Using an initializer class:

```java
public class MyWebAppInitializer implements WebApplicationInitializer {
 
    @Override
    public void onStartup(ServletContext container) {
        AnnotationConfigWebApplicationContext context
          = new AnnotationConfigWebApplicationContext();
        context.setConfigLocation("com.baeldung");
 
        container.addListener(new ContextLoaderListener(context));
 
        ServletRegistration.Dynamic dispatcher = container
          .addServlet("dispatcher", new DispatcherServlet(context));
         
        dispatcher.setLoadOnStartup(1);
        dispatcher.addMapping("/");
    }
}

@EnableWebMvc
@Configuration
public class ClientWebConfig implements WebMvcConfigurer { 
   @Bean
   public ViewResolver viewResolver() {
      InternalResourceViewResolver bean
        = new InternalResourceViewResolver();
      bean.setViewClass(JstlView.class);
      bean.setPrefix("/WEB-INF/view/");
      bean.setSuffix(".jsp");
      return bean;
   }
}
```

In Spring Boot, by comparison:

```
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```

Spring Boot will look at the dependencies, properties, and beans that exist in the application and enable configuration based on these. 

----

## References

1. https://www.baeldung.com/spring-vs-spring-boot