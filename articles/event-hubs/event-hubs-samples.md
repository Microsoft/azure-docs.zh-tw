---
title: Azure 事件中樞範例 | Microsoft Docs
description: Azure 事件中樞範例
services: event-hubs
documentationcenter: na
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.assetid: ''
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/17/2018
ms.author: shvija
ms.openlocfilehash: fbde6e5a5ed053d6c151b3af25535c397a496ef4
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40005329"
---
# <a name="event-hubs-samples"></a>事件中樞範例 
您可以在 [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples) 上尋找事件中樞範例。 這些範例示範 [Azure 事件中樞](/azure/event-hubs/)中的主要功能。 本主題分類及描述可用的範例與每個範例的連結。

## <a name="net-samples"></a>.NET 範例

| 範例名稱 | 說明 | 
| ----------- | ----------- | 
| [SampleSender](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/SampleSender) | 這個範例示範如何撰寫 .NET Core 主控台應用程式，以將一組事件傳送到事件中樞。 |
| [SampleEHReceiver](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/SampleEphReceiver) | 這個範例示範如何撰寫 .NET Core 主控台應用程式，以使用「事件處理器主機」程式庫從事件中樞接收一組事件。  | 

## <a name="java-samples"></a>Java 範例

| 範例名稱 | 說明 | 
| ----------- | ----------- | 
| [SendBatch](https://github.com/Azure/azure-event-hubs/tree/master/samples/Java/Basic/SendBatch)  | 這個範例說明如何將事件批次內嵌到事件中樞。 | 
| [SimpleSend](https://github.com/Azure/azure-event-hubs/tree/master/samples/Java/Basic/SimpleSend) | 這個範例說明如何將事件內嵌到事件中樞。 |
| [AdvanceSendOptions](https://github.com/Azure/azure-event-hubs/blob/master/samples/Java/Basic/AdvancedSendOptions) | 這個範例說明各種可供事件中樞用於內嵌事件的選項。 |
| [ReceiveByDateTime](https://github.com/Azure/azure-event-hubs/blob/master/samples/Java/Basic/ReceiveByDateTime) | 這個範例說明如何使用特定日期時間位移，從事件中樞分割區接收事件。 |
| [ReceiveUsingOffset](https://github.com/Azure/azure-event-hubs/blob/master/samples/Java/Basic/ReceiveUsingOffset) | 這個範例說明如何使用特定日期位移，從事件中樞分割區接收事件。 |  
| [ReceiveUsingSequenceNumber](https://github.com/Azure/azure-event-hubs/blob/master/samples/Java/Basic/ReceiveUsingSequenceNumber) | 這個範例說明如何使用序號，從事件中樞分割區進行接收。 |   
| [EventProcessorSample](https://github.com/Azure/azure-event-hubs/blob/master/samples/Java/Basic/EventProcessorSample) |這個範例說明如何使用事件處理器主機從事件中樞接收事件，該主機可提供自動分割區選取和跨越多個並行接收端的容錯移轉。 | 
| [AutoScaleOnIngress](https://github.com/Azure/azure-event-hubs/blob/master/samples/Java/Benchmarks/AutoScaleOnIngress) | 這個範例說明事件中樞如何才能在高負載的情況下自動相應增加。 這個範例傳送事件的速率僅超過事件中樞已設定的速率，因而導致事件中樞相應增加。 | 
| [IngressBenchmark](https://github.com/Azure/azure-event-hubs/blob/master/samples/Java/Benchmarks/IngressBenchmark) | 這個範例可讓您測量輸入速率。 | 

## <a name="next-steps"></a>後續步驟
您可以參閱下列文章，深入了解事件中樞：

- [事件中心概觀](event-hubs-what-is-event-hubs.md)
- [事件中樞功能](event-hubs-features.md)
- [事件中樞常見問題集](event-hubs-faq.md)