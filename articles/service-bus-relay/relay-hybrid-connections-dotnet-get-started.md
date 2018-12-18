---
title: 在 .NET 中開始使用 Azure 轉送混合式連線 Websocket | Microsoft Docs
description: 為 Azure 轉送混合式連線 Websocket 撰寫 C# 主控台應用程式。
services: service-bus-relay
documentationcenter: .net
author: spelluru
manager: timlt
editor: ''
ms.assetid: d1386900-b942-4abf-acfc-38d2ef826253
ms.service: service-bus-relay
ms.devlang: tbd
ms.topic: get-started-article
ms.tgt_pltfrm: dotnet
ms.workload: na
ms.date: 12/15/2017
ms.author: spelluru
ms.openlocfilehash: 1ed401f6175d7ebea83a888898221d345791bc34
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43697953"
---
# <a name="get-started-with-relay-hybrid-connections-websockets-in-net"></a>在 .NET 中開始使用轉送混合式連線 Websocket
[!INCLUDE [relay-selector-hybrid-connections](../../includes/relay-selector-hybrid-connections.md)]

本教學課程會介紹 [Azure 轉送混合式連線](relay-what-is-it.md#hybrid-connections)。 了解如何使用 Microsoft .NET 來建立用戶端應用程式，以將訊息傳送至對應的接聽程式應用程式。 

## <a name="what-will-be-accomplished"></a>將會完成的工作
混合式連線需要同時用到用戶端元件和伺服器元件。 在本教學課程中，您會完成下列步驟來建立兩個主控台應用程式：

1. 使用 Azure 入口網站建立轉送命名空間。
2. 使用 Azure 入口網站，在該命名空間中建立混合式連線。
3. 撰寫伺服器 (接聽端) 主控台應用程式來接收訊息。
4. 撰寫用戶端 (傳送端) 主控台應用程式來傳送訊息。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您需要下列必要條件：

* [Visual Studio 2015 或更新版本](http://www.visualstudio.com)。 本教學課程中的範例使用 Visual Studio 2017。
* Azure 訂用帳戶。

[!INCLUDE [create-account-note](../../includes/create-account-note.md)]

## <a name="1-create-a-namespace-by-using-the-azure-portal"></a>1.使用 Azure 入口網站建立命名空間
如果您已經建立轉送命名空間，請移至[使用 Azure 入口網站建立混合式連線](#2-create-a-hybrid-connection-using-the-azure-portal)。

[!INCLUDE [relay-create-namespace-portal](../../includes/relay-create-namespace-portal.md)]

## <a name="2-create-a-hybrid-connection-by-using-the-azure-portal"></a>2.使用 Azure 入口網站建立混合式連線
如果您已經建立混合式連線，請移至[建立伺服器應用程式](#3-create-a-server-application-listener)。

[!INCLUDE [relay-create-hybrid-connection-portal](../../includes/relay-create-hybrid-connection-portal.md)]

## <a name="3-create-a-server-application-listener"></a>3.建立伺服器應用程式 (接聽程式)
在 Visual Studio 中，撰寫 C# 主控台應用程式以接聽並接收來自轉送的訊息。

[!INCLUDE [relay-hybrid-connections-dotnet-get-started-server](../../includes/relay-hybrid-connections-dotnet-get-started-server.md)]

## <a name="4-create-a-client-application-sender"></a>4.建立用戶端應用程式 (傳送者)
在 Visual Studio 中，撰寫 C# 主控台應用程式以將訊息傳送至轉送。

[!INCLUDE [relay-hybrid-connections-dotnet-get-started-client](../../includes/relay-hybrid-connections-dotnet-get-started-client.md)]

## <a name="5-run-the-applications"></a>5.執行應用程式
1. 執行伺服器應用程式。
2. 執行用戶端應用程式並輸入一些文字。
3. 確定伺服器應用程式主控台有顯示用戶端應用程式中所輸入的文字。

![執行應用程式](./media/relay-hybrid-connections-dotnet-get-started/running-applications.png)

恭喜您，您已建立端對端混合式連線應用程式！

## <a name="next-steps"></a>後續步驟

* [轉送常見問題集](relay-faq.md)
* [建立命名空間](relay-create-namespace-portal.md)
* [開始使用 Node](relay-hybrid-connections-node-get-started.md)

