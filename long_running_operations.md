---
title: Long-running operatons
description: An overview of the Azure SDK for Java concepts related to long-running operatons
ms.date: 11/23/2020
ms.topic: conceptual
ms.custom: devx-track-java
---

# Long-running operations

Certain operations on Azure may take extended amounts of time to complete, that push them outside of the standard HTTP style of quick request / response flow. For example, copying data from a source URL to a Storage blob or training a model to recognize forms are operations that may take a few seconds to several minutes. Such operations are referred to as Long-Running Operations, and are often abbreviated as 'LRO'. An LRO may take seconds, minutes, hours, days, or longer to complete, depending on the operation requested and the process that must be performed on the server side.

In the Java client libraries for Azure, a convention exists that all long-running operations begin with the `begin` prefix. Whenever a `begin` prefix is encountered, developers are encouraged to understand that this operation will be long-running, and that the means of interaction with this operation will be slightly different that the usual request / response flow. Along with the `begin` prefix, the return type from the operation will also be different than usual, to enable the full range of long-running operation functionality. As with most things in the Azure SDK for Java, there are both synchronous and asynchronous APIs for long-running operations:

* In synchronous clients, long-running operations will return a `SyncPoller` instance.
* In asynchronous clients, long-running operations will return a `PollerFlux` instance.

Both `SyncPoller` and `PollerFlux` are the client-side abstractions intended to simplify the interaction with long-running server-side operations. The remainder of this document outlines best practices when working with these types.

## Synchronous long-running operations

> TODO

## Asynchronous long-running operations

The code below shows how the `PollerFlux` API allows users to observe a long-running operation. It is important to note that in asynchronous APIs the network calls happens in a different thread than the main-thread that calls subscribe(). This means that the main-thread may terminate before the result is available. It is up to the developer to ensure that the application does not exit before the async operation has had time to complete.

In the async API, a `PollerFlux` will be returned immediately, but the long-running operation itself will not commence until a subscription is made to the returned `PollerFlux`. This is the same as how all `Flux`-based APIs operate. Shown below is an example of an async long-running operation:

```java
asyncClient.beginUploadFromUri(...)
    .subscribe(response -> System.out.println("Status of long running upload operation: " + response.getStatus()));
```

In this example, we will get intermittent status updates on the long-running operation, from which we can determine if it is still operating in the expected fashion. In this case the code is simply being printed to the console, but a better implementation would make relevant error handling decisions based on this status.

If we are not interested in the intermediate status updates and just want to get notified of the final result once it has arrived, we may do the following in our code:

```java
asyncClient.beginUploadFromUri("{form-url")
    .last()
    .flatMap(response -> {
        if (LongRunningOperationStatus.SUCCESSFULLY_COMPLETED == response.getStatus()) {
            return response.getFinalResult();
        }
        return Mono.error(new IllegalStateException("Polling completed unsuccessfully with status: "+ response.getStatus()));
    })
    .subscribe(
        finalResult -> processFormPages(finalResult),
        ex -> countDownLatch.countDown(),
        () -> countDownLatch.countDown());
```

In this code we retrieve the final result of the long-running operation by calling `last()`. This tells the `PollerFlux` that we want to wait for all the polling to complete, at which point the long-running operation has reached a terminal state, and its status can be inspected to determine the outcome. If the poller indicates that the long-running operation has completed successfully, the final result can be retrieved and passed on to the consumer in the subscribe call.

The following sample code using `last()` operator to wait for all polling to complete, then retrieve the final result if LRO succeeded.
