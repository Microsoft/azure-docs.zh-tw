---
title: Azure 轉送連接埠設定 | Microsoft Docs
description: Azure 轉送連接埠值的相關詳細資料。
services: service-bus-relay
documentationcenter: na
author: spelluru
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-bus-relay
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/26/2018
ms.author: spelluru
ms.openlocfilehash: 3ef08cfc94a029f97250578e9b0366a18770c809
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43696731"
---
# <a name="azure-relay-port-settings"></a>Azure 轉送連接埠設定

下表說明 Azure 轉送的連接埠值所需的設定。

## <a name="hybrid-connections"></a>混合式連線

混合式連線會在具有 SSL 的連接埠 443 上使用 WebSockets 作為基礎傳輸機制，而僅使用 **HTTPS**。 

## <a name="wcf-relays"></a>WCF 轉送
  
|繫結|傳輸安全性|Port|  
|-------------|------------------------|----------|  
|[BasicHttpRelayBinding 類別](/dotnet/api/microsoft.servicebus.basichttprelaybinding) (用戶端)|是|HTTPS| 
|" |否|HTTP|  
|[BasicHttpRelayBinding 類別](/dotnet/api/microsoft.servicebus.basichttprelaybinding) (服務)|無論是|9351/HTTP|  
|[NetEventRelayBinding 類別](/dotnet/api/microsoft.servicebus.neteventrelaybinding) (用戶端)|是|9351/HTTPS|  
|" |否|9350/HTTP|  
|[NetEventRelayBinding 類別](/dotnet/api/microsoft.servicebus.neteventrelaybinding) (服務)|無論是|9351/HTTP|  
|[NetTcpRelayBinding 類別](/dotnet/api/microsoft.servicebus.nettcprelaybinding) (用戶端/服務)|無論是|5671/9352/HTTP (如果使用混合式則為 9352/9353)|  
|[NetOnewayRelayBinding 類別](/dotnet/api/microsoft.servicebus.netonewayrelaybinding) (用戶端)|是|9351/HTTPS|  
|" |否|9350/HTTP|  
|[NetOnewayRelayBinding 類別](/dotnet/api/microsoft.servicebus.netonewayrelaybinding) (服務)|無論是|9351/HTTP|  
|[WebHttpRelayBinding 類別](/dotnet/api/microsoft.servicebus.webhttprelaybinding) (用戶端)|是|HTTPS|  
|" |否|HTTP|  
|[WebHttpRelayBinding 類別](/dotnet/api/microsoft.servicebus.webhttprelaybinding) (服務)|無論是|9351/HTTP|  
|[WS2007HttpRelayBinding 類別](/dotnet/api/microsoft.servicebus.ws2007httprelaybinding) (用戶端)|是|HTTPS|  
|" |否|HTTP|  
|[WS2007HttpRelayBinding 類別](/dotnet/api/microsoft.servicebus.ws2007httprelaybinding) (服務)|無論是|9351/HTTP|

## <a name="next-steps"></a>後續步驟
若要深入了解 Azure 轉送，請造訪下列連結：
* [什麼是 Azure 轉送？](relay-what-is-it.md)
* [轉送常見問題集](relay-faq.md)