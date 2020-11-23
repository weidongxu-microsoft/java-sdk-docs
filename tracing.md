---
title: Tracing
description: An overview of the Azure SDK for Java concepts related to tracing
ms.date: 11/23/2020
ms.topic: conceptual
ms.custom: devx-track-java
---

# Tracing

The Azure SDK for Java enables tracing in all client libraries by simply including a dependency on the [OpenTelemetery](https://opentelemetry.io/)-based `azure-core-tracing-opentelemetry` [plugin](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/core/azure-core-tracing-opentelemetry#azure-tracing-opentelemetry-client-library-for-java). OpenTelemetry is a popular open-source observability framework for generating, capturing, and collecting telemetry data for cloud-native software.

## Key concepts

### Trace

A trace is a tree of spans showing the path of work through a system. A trace on its own is distinguishable by a unique 16 byte sequence called a TraceID.

### Span

A span represents a single operation in a trace. A span could be representative of an HTTP request, a remote procedure call (RPC), a database query, or even the path that a code takes.

## Configure Tracer

To enable tracing for the client libraries, developers will have to add the `azure-core-tracing-opentelemetry` plugin and `opentelemetry-sdk` package dependency to their application.

```xml
<dependency>
  <groupId>com.azure</groupId>
  <artifactId>azure-core-tracing-opentelemetry</artifactId>
  <version>1.0.0-beta.6</version>
</dependency>

<dependency>
  <groupId>io.opentelemetry</groupId>
  <artifactId>opentelemetry-sdk</artifactId>
  <version>0.6.0</version>
</dependency>
```

By adding this dependency, tracing will be enabled and traces will be created and included with all HTTP requests. There will be no integration however with any incoming parent span, and it is still required to specify where these tracer spans should be exported to. Let's do that now.

### Customizing Tracer spans:

Users can additionally create a root span in their application and pass it down to the client calls for encapsulating all the outgoing requests in the application. This can be done by using the `Context` parameter on the client methods.

```java
Context traceContext = new Context(PARENT_SPAN_KEY, span);
appConfigClient.setConfigurationSettingWithResponse(new ConfigurationSetting().setKey("hello").setValue("world"), true, traceContext);
```

>In case of no parent span provided, a new parent span will be created to encapsulate all the client libraries outgoing requests.

Each client call results in two spans, convenience layer span and outgoing HTTP request span.

#### Tracer Span Attributes

In addition to OpenTelemetry's required standard attributes mentioned [here](https://github.com/open-telemetry/opentelemetry-specification/blob/e9340d74f1ba0b651b3581d6bd5df6a92b772e18/semantic-conventions.md), client libraries annotates the spans with below mentioned attributes:

* `az.namespace`: Microsoft resource provider [namespaces](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/azure-services-resource-providers) mapped to Azure services.
* `x-ms-request-id`: The unique identifier for the request.
* `span.kind`: Describes the relationship between the Span, its parents, and its children in a Trace.
* `span.status.message`: Represents the status of a finished Span.
* `span.status.code`: Represents the status code of a finished Span.

Additional metadata about the operation being performed is captured in the span names. The HTTP span names are set to the URI path value and the library method invocation span is of the form `<namespace qualified type>.<method name>`.

For example, an App Configuration client request to set Configuration setting i.e `appConfigClient.setConfigurationSettingWithResponse(new ConfigurationSetting().setKey("hello").setValue("world")` will result two spans:

* Library method span named `AppConfig.setKey`.
* HTTP outgoing request span named `/kv/hello`.

## Configure Exporter:

User applications can export traces to different distributed tracing stores (such as Zipkin, Jeager, Stackdriver Trace).

```java
// 1. Configure exporter to export traces to Jaeger.
ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 14250).usePlaintext().build();
JaegerGrpcSpanExporter exporter = JaegerGrpcSpanExporter.newBuilder()
    .setChannel(channel)
    .setServiceName("Sample")
    .setDeadline(0)
    .build();
TracerSdkFactory tracerSdkFactory = (TracerSdkFactory) OpenTelemetry.getTracerFactory();
tracerSdkFactory.addSpanProcessor(SimpleSpansProcessor.newBuilder(exporter).build());
```

## Configure Application insights

User applications can export traces to [Application Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) dashboard by adding the Java [in-process agent](https://github.com/microsoft/ApplicationInsights-Java/releases/download/3.0.0-PREVIEW.5/applicationinsights-agent-3.0.0-PREVIEW.5.jar) and the [azure-core-tracing-opentelemetry](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/core/azure-core-tracing-opentelemetry#azure-tracing-opentelemetry-client-library-for-java) package to their project.

More information about attaching the Java codeless in-process agent and Application Insights can be found [here](https://docs.microsoft.com/en-us/azure/azure-monitor/app/java-in-process-agent).
