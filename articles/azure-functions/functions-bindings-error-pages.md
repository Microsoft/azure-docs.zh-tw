---
title: Azure 函式錯誤處理指引 | Microsoft Docs
description: 提供當函式執行時所發生錯誤的處理之一般指引，以及繫結特定錯誤主題的連結。
services: functions
cloud: ''
documentationcenter: ''
author: ggailey777
manager: jeconnoc
ms.assetid: ''
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 02/01/2018
ms.author: glenga; cfowler
ms.openlocfilehash: 5a8dae73c164b319b4c291685deff402f9798364
ms.sourcegitcommit: af60bd400e18fd4cf4965f90094e2411a22e1e77
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44091829"
---
# <a name="azure-functions-error-handling"></a>Azure 函式錯誤處理

本主題提供您的函式在執行時所發生錯誤的處理之一般指引。 它也提供描述，可能會發生的繫結特定錯誤之主題的連結。 

## <a name="handing-errors-in-functions"></a>處理函式中的錯誤
[!INCLUDE [bindings errors intro](../../includes/functions-bindings-errors-intro.md)]

 
## <a name="binding-error-codes"></a>繫結錯誤碼

在與 Azure 服務整合時，您可能會遇到來自基礎服務 API 發出的錯誤。 這些服務的錯誤碼文件之連結，可以在下列觸發程序和繫結參考主題的＜例外狀況和傳回碼＞小節中找到：

+ [Azure Cosmos DB](functions-bindings-cosmosdb.md#exceptions-and-return-codes)

+ [Blob 儲存體](functions-bindings-storage-blob.md#exceptions-and-return-codes)

+ [事件中樞](functions-bindings-event-hubs.md#exceptions-and-return-codes)

+ [通知中樞](functions-bindings-notification-hubs.md#exceptions-and-return-codes)

+ [佇列儲存體](functions-bindings-storage-queue.md#exceptions-and-return-codes)

+ [服務匯流排](functions-bindings-service-bus.md#exceptions-and-return-codes)

+ [資料表儲存體](functions-bindings-storage-table.md#exceptions-and-return-codes)
