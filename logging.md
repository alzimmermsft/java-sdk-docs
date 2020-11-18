# Logging

The Azure client libraries for Java log using [SLF4J](https://www.slf4j.org/), allowing for applications using these libraries to use their preferred logging framework. More details on logging using SLF4J can be found 
[in the SLF4J manual](https://www.slf4j.org/manual.html). 

## Configuring Logging

By default, logging should be configured using an SLF4J-supported logging framework. This starts by including a [relevant SLF4J logging implementation as a dependency from your project](http://www.slf4j.org/manual.html#projectDep), but then continues onward to configuring your logger to work as necessary in your environment (such as setting log levels, configuring which classes do and do not log, etc). Some examples are provided below, but for more detail, refer to the documentation for your chosen logging framework.

### Fall-Back Logging (For Temporary Debugging)

Logging, as already stated, uses SLF4J, but there is a fall-back, default logger built into the Azure client libraries for Java for circumstances where an application is deployed, and logging is required, but it is not possible to redeploy the application with an SLF4J logger included. To enable this logger, you must firstly be certain that there exists no SLF4J logger (as this will take precedence), and then set the `AZURE_LOG_LEVEL` environment variable. Refer to the following table for the allowed values this environment variable:

| Log Level              | Allowed Environment Variable Values     |
|------------------------|-----------------------------------------|
| VERBOSE                | "verbose", "debug"                      | 
| INFORMATIONAL          | "info", "information", "informational"  |  
| WARNING                | "warn", "warning"                       |  
| ERROR                  | "err", "error"                          |

## Use `ClientLogger` for logging information.

Here is the code to use `ClientLogger`

```java
   ClientLogger logger = new ClientLogger("$ClassName"); // Use the string of class name or the class type.
   logger.info("I am info");
```

## Configuration Examples

### Use log4j logging framework

For more information related to log4j, please refer [here](http://logging.apache.org/log4j/1.2/).

**Adding maven dependencies**

```xml
<!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-log4j12 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>[1.0,)</version> <!-- Version number 1.0 and above -->
</dependency>
```

**Using property file**

Usually, to enable log4j in property file, you can create `log4j.properties` under `./src/main/resource` directory of your project.

Log4j example:

```properties
log4j.rootLogger=INFO, A1
log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%m%n
log4j.logger.com.azure.core=ERROR
```

**Using xml**

Usually, to enable log4j in property file, you can create `log4j.xml` under `./src/main/resource` directory of your project.

log4j example:

```xml
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
<log4j:configuration debug="true" xmlns:log4j='http://jakarta.apache.org/log4j/'>

    <appender name="console" class="org.apache.log4j.ConsoleAppender">
        <param name="Target" value="System.out"/>
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%m%n" />
        </layout>
    </appender>
    <logger name="com.azure.core">
        <level value="ERROR" />
        <appender-ref ref="console" />
    </logger>

    <root>
        <level value="info" />
        <appender-ref ref="console" />
    </root>

</log4j:configuration>
```

### Use log4j2 logging framework

For more information related to log4j2, please refer [here](https://logging.apache.org/log4j/2.x/manual/configuration.html).

**Adding maven dependencies**

```xml
<!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-slf4j-impl -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>[2.0,)</version> <!-- Version number 2.0 and above -->
</dependency>
```

**Using property file**

Usually, to enable log4j2 in property file, you can create `log4j2.properties` under `./src/main/resource` directory of your project.

Log4j2 example:

```properties
appender.console.type = Console
appender.console.name = STDOUT
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = %msg%n
logger.app.name=com.azure.core
logger.app.level=ERROR

rootLogger.level = info
rootLogger.appenderRefs = stdout
rootLogger.appenderRef.stdout.ref = STDOUT
```

**Using xml**

Usually, to enable log4j2 in property file, you can create `log4j2.xml` under `./src/main/resource` directory of your project.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="INFO">
    <Appenders>
        <Console name="console" target="SYSTEM_OUT">
            <PatternLayout
                pattern="%msg%n" />
        </Console>
    </Appenders>
    <Loggers>
        <Logger name="com.azure.core" level="error" additivity="true">
            <appender-ref ref="console" />
        </Logger>
        <Root level="info" additivity="false">
            <appender-ref ref="console" />
        </Root>
     </Loggers>
</Configuration>
```

### Use logback logging framework
To enable logback logging in config file, create a file called `logback.xml` under `./src/main/resources` directory of your project. For more information relate to logback, please refer [here](http://logback.qos.ch/manual/configuration.html)

**Adding maven dependencies**

```xml
<!-- https://mvnrepository.com/artifact/ch.qos.logback/logback-classic -->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>[0.2.5,)</version> <!-- Version number 0.2.5 and above -->
</dependency>
```

```xml
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%message%n</pattern>
    </encoder>
  </appender>

  <logger name="com.azure.core" level="ERROR" />

  <root level="INFO">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

### Use logback logging framework in a Spring Boot application

[Logback](https://logback.qos.ch/manual/introduction.html) is one of the popular logging frameworks.
To enable logback logging, create a file called `logback.xml` under `./src/main/resources` directory of your project.
This file will contain the logging configurations to customize your logging needs. More information 
on configuring `logback.xml` can be found [here](https://logback.qos.ch/manual/configuration.html). 

A simple logback configuration to log to console can be configured as follows:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <appender name="Console"
    class="ch.qos.logback.core.ConsoleAppender">
    <layout class="ch.qos.logback.classic.PatternLayout">
      <Pattern>
        %black(%d{ISO8601}) %highlight(%-5level) [%blue(%t)] %blue(%logger{100}): %msg%n%throwable
      </Pattern>
    </layout>
  </appender>

  <root level="INFO">
    <appender-ref ref="Console" />
  </root>
</configuration>
```

Spring looks at this file for various configurations including logging. You can configure your application to read logback configurations from any file. So, this is where you will link your `logback.xml` to your spring application. Add the following line to do so:

Create another file called `application.properties` under the same directory `./src/main/resources`.

```properties
logging.config=classpath:logback.xml
```

To configure logging to a file which is rolled over after each hour and archived in gzip format:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <property name="LOGS" value="./logs" />
  <appender name="RollingFile" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${LOGS}/spring-boot-logger.log</file>
    <encoder
      class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
      <Pattern>%d %p %C{1.} [%t] %m%n</Pattern>
    </encoder>

    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- rollover hourly and gzip logs -->
      <fileNamePattern>${LOGS}/archived/spring-boot-logger-%d{yyyy-MM-dd-HH}.log.gz</fileNamePattern>
    </rollingPolicy>
  </appender>

  <!-- LOG everything at INFO level -->
  <root level="info">
    <appender-ref ref="RollingFile" />
  </root>
</configuration>
```
