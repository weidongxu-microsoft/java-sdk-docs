---
title: Azure SDK Context
description: An overview of the Azure SDK Context type
ms.date: 11/23/2020
ms.topic: conceptual
ms.custom: devx-track-java
---

> **NOTE:** This document is in **draft form**, as there are upcoming changes coming in Project Reactor, and it would be unwise to give advice before these changes hit.

# Azure SDK Context

The Azure SDK for Java [Context](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/core/azure-core/src/main/java/com/azure/core/util/Context.java) type can be viewed as a key-value map that can be passed into a variation of every API call in all client libraries to enable per-request customization. The means in which a `Context` is supplied to a method differs between sync and async APIs:

* In sync APIs, the `Context` type is accepted into synchronous APIs as a method parameter, typically on methods that end in `WithResponse`.
* In async APIs, the `Context` type is supplied via the [Project Reactor subscriberContext]
