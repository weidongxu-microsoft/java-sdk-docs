# Use logback logging framework
To enable logback logging, first, you must include the logback library as a dependency and then create a file called `logback.xml` under `./src/main/resources` directory of your project. For more information related to logback, please refer [here](http://logback.qos.ch/manual/configuration.html)

## Adding maven dependencies

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

## Add `logback.xml` to your project

[Logback](https://logback.qos.ch/manual/introduction.html) is one of the popular logging frameworks.
To enable logback logging, create a file called `logback.xml` under `./src/main/resources` directory of your project.
This file will contain the logging configurations to customize your logging needs. More information 
on configuring `logback.xml` can be found [here](https://logback.qos.ch/manual/configuration.html). 

### Console logging
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

### Log to a file with log rotation enabled 
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

### Spring applications

If you are using Spring framework, Spring looks at `application.properties` file for various configurations including logging. You can configure your application to read logback configurations from any file. So, this is where you can link your `logback.xml` to your Spring application. Add the following line to do so:

Create another file called `application.properties` under the same directory `./src/main/resources`.

```properties
logging.config=classpath:logback.xml
```
