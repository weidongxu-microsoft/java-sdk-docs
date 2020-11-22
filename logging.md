# Logging

The Azure client libraries for Java log using [SLF4J](https://www.slf4j.org/) interface, allowing for applications using Azure libraries to use their preferred logging framework. More details on logging using SLF4J, please refer to [SLF4J manual](https://www.slf4j.org/manual.html). 

## Configuring Logging

By default, logging should be configured using an SLF4J-supported logging framework. This starts by including a [relevant SLF4J logging implementation as a dependency from your project](http://www.slf4j.org/manual.html#projectDep), but then continues onward to configuring your logger to work as necessary in your environment (such as setting log levels, configuring which classes do and do not log, etc). Some examples are provided below, but for more detail, refer to the documentation for your chosen logging framework.

## Use `ClientLogger` for logging information.

If you are using any new Azure client libraries, it will include a logger class called `ClientLogger` that uses any available SLF4J implementation or logs to console if SLF4J implementation is not available. Here is a sample code to use `ClientLogger`:

```java
   ClientLogger logger = new ClientLogger("$ClassName"); // Use the string of class name or the class type.
   logger.info("I am info");
```

## Configuration Examples
- [Logback](logback.md)
- [Log4J](log4j.md)
- [Fallback logging using DefaultLogger](default_logger.md)
