---
title: AsyncService type
description: Provides ability to perform async operations from business logic code.
author: robinarh
ms.date: 08/01/2017
audience: Developer
ms.reviewer: rhaertle
ms.search.region: Global
ms.author: rhaertle
---

# AsyncService type

[!include [banner](../../../../includes/banner.md)]

Provides ability to perform async operations from business logic code.

### Hierarchy

AsyncService

## Index

### Method list

* [all](services-business-logic-services-iasyncservice.md#all)
* [defer](services-business-logic-services-iasyncservice.md#defer)

## Methods

### all

`all(...args: any [ ]): Promise <any [ ]>`

#### Parameters

| Name | Type | Description |
| ---- | ---- | ----------- |
| ...args|any [ ]||

#### Returns Promise &lt;any [ ]&gt;

### defer

`defer <>(): [Deferred](defer-ideferred.md) <>`

Creates a deferred object that can be used to return a promise from event handlers (where applicable) and resolve/reject them asynchronously.

#### Returns [Deferred](defer-ideferred.md) &lt;T&gt;


[!INCLUDE[footer-include](../../../../../../includes/footer-banner.md)]