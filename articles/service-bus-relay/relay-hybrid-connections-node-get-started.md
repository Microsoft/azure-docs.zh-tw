---
title: 在 Node 中開始使用 Azure 轉送混合式連線 Websocket | Microsoft Docs
description: 為 Azure 轉送混合式連線 Websocket 撰寫 Node.js 主控台應用程式
services: service-bus-relay
documentationcenter: node
author: spelluru
manager: timlt
editor: ''
ms.assetid: e44e4867-3cf3-46be-8f8a-7671e2013bc4
ms.service: service-bus-relay
ms.devlang: tbd
ms.topic: get-started-article
ms.tgt_pltfrm: node
ms.workload: na
ms.date: 05/02/2018
ms.author: spelluru
ms.openlocfilehash: 25e9095561f72583bad86aa96b64a412e0983ab6
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43702352"
---
# <a name="get-started-with-relay-hybrid-connections-websockets-in-node"></a>在 Node 中開始使用轉送混合式連線 Websocket

[!INCLUDE [relay-selector-hybrid-connections](../../includes/relay-selector-hybrid-connections.md)]

本教學課程提供 [Azure 轉送混合式連線](relay-what-is-it.md#hybrid-connections) Websocket 功能的簡介，並示範如何使用 Node.js 來建立用戶端應用程式，以將 Websocket 訊息傳送至對應的接聽端應用程式。

## <a name="what-will-be-accomplished"></a>將會完成的工作

因為混合式連線同時需要用戶端和伺服器元件，請在本教學課程中建立兩個主控台應用程式。 步驟如下：

1. 使用 Azure 入口網站建立轉送命名空間。
2. 使用 Azure 入口網站建立混合式連線。
3. 撰寫伺服器主控台應用程式來接收訊息。
4. 撰寫用戶端主控台應用程式來傳送訊息。

## <a name="prerequisites"></a>必要條件

1. [Node.js](https://nodejs.org/en/).
2. Azure 訂用帳戶。

[!INCLUDE [create-account-note](../../includes/create-account-note.md)]

## <a name="1-create-a-namespace-using-the-azure-portal"></a>1.使用 Azure 入口網站建立命名空間

如果您已經建立轉送命名空間，請跳至[使用 Azure 入口網站建立混合式連線](#2-create-a-hybrid-connection-using-the-azure-portal)一節。

[!INCLUDE [relay-create-namespace-portal](../../includes/relay-create-namespace-portal.md)]

## <a name="2-create-a-hybrid-connection-using-the-azure-portal"></a>2.使用 Azure 入口網站建立混合式連線

如果您已經建立混合式連線，請跳至[建立伺服器應用程式](#3-create-a-server-application-listener)一節。

[!INCLUDE [relay-create-hybrid-connection-portal](../../includes/relay-create-hybrid-connection-portal.md)]

## <a name="3-create-a-server-application-listener"></a>3.建立伺服器應用程式 (接聽程式)

為了接聽並接收來自轉送的訊息，請撰寫 Node.js 主控台應用程式。

[!INCLUDE [relay-hybrid-connections-node-get-started-server](../../includes/relay-hybrid-connections-node-get-started-server.md)]

## <a name="4-create-a-client-application-sender"></a>4.建立用戶端應用程式 (傳送者)

為了傳送訊息到轉送，請撰寫 Node.js 主控台應用程式。

[!INCLUDE [relay-hybrid-connections-node-get-started-client](../../includes/relay-hybrid-connections-node-get-started-client.md)]

## <a name="5-run-the-applications"></a>5.執行應用程式

1. 執行伺服器應用程式：從 Node.js 命令提示字元中，鍵入 `node listener.js`。
2. 執行用戶端應用程式：從 Node.js 命令提示字元中，鍵入 `node sender.js`，然後輸入一些文字。
3. 確定伺服器應用程式主控台有將用戶端應用程式中所輸入的文字輸出。

![執行應用程式](./media/relay-hybrid-connections-node-get-started/running-applications.png)

恭喜您，您已經使用 Node.js 建立端對端混合式連線應用程式！

## <a name="next-steps"></a>後續步驟

* [轉送常見問題集](relay-faq.md)
* [建立命名空間](relay-create-namespace-portal.md)
* [開始使用 .NET](relay-hybrid-connections-dotnet-get-started.md)
* [開始使用 Node](relay-hybrid-connections-node-get-started.md)

