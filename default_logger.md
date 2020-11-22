# Default logger (For Temporary Debugging)

Azure client libraries use SLF4J for [logging](logging.md), but there is a fallback, default logger built into Azure client libraries for Java for circumstances where an application is deployed, and logging is required, but it is not possible to redeploy the application with an SLF4J logger included. To enable this logger, you must firstly be certain that there exists no SLF4J logger (as this will take precedence), and then set the `AZURE_LOG_LEVEL` environment variable. Refer to the following table for the allowed values for this environment variable:

| Log Level              | Allowed Environment Variable Values     |
|------------------------|-----------------------------------------|
| VERBOSE                | "verbose", "debug"                      | 
| INFORMATIONAL          | "info", "information", "informational"  |  
| WARNING                | "warn", "warning"                       |  
| ERROR                  | "err", "error"                          |

After the environment variable is set, please restart the application for the environment variable to take effect. Note that enabling this logger will log to the console and does not provide additional customization capabilities like rollover, logging to file, etc. that an SLF4J implementation will provide. To turn the logging off again, just remove the environment variable and restart the application.