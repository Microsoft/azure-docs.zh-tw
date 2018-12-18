---
title: Azure Service Fabric 搭配 API 管理概觀 | Microsoft Docs
description: 本文是使用「Azure API 管理」作為 Service Fabric 應用程式閘道的簡介。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: ''
ms.assetid: 96176149-69bb-4b06-a72e-ebbfea84454b
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/22/2017
ms.author: vturecek
ms.openlocfilehash: 6bf7ea90bb5351411984110fd8fb05c2f8cb0650
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34205156"
---
# <a name="service-fabric-with-azure-api-management-overview"></a>Service Fabric 搭配 Azure API 管理概觀

雲端應用程式通常需要前端閘道來為使用者、裝置或其他應用程式提供單一輸入點。 在 Service Fabric 中，閘道可以是任何無狀態服務 (例如 [ASP.NET Core 應用程式](service-fabric-reliable-services-communication-aspnetcore.md))，或是為流量輸入設計的另一個服務 (例如[事件中樞](https://docs.microsoft.com/azure/event-hubs/)、[IoT 中樞](https://docs.microsoft.com/azure/iot-hub/)或 [Azure API 管理](https://docs.microsoft.com/azure/api-management/))。

本文是使用「Azure API 管理」作為 Service Fabric 應用程式閘道的簡介。 「API 管理」直接與 Service Fabric 整合，可讓您將具有一組豐富路由規則的 API 發佈至後端 Service Fabric 服務。 

## <a name="architecture"></a>架構
常見的 Service Fabric 架構會使用單頁 Web 應用程式，此應用程式會對公開 HTTP API 的後端服務發出 HTTP 呼叫。 [Service Fabric 快速入門範例應用程式](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)說明此架構的範例。

在此案例中，無狀態 Web 服務會作為進入 Service Fabric 應用程式的閘道。 此方法需要您撰寫一個 Web 服務，此服務必須能夠作為 HTTP 要求的 Proxy 來將這些要求傳送到後端服務，如下圖所示：

![Service Fabric 搭配 Azure API 管理拓撲概觀][sf-web-app-stateless-gateway]

隨著應用程式越來越複雜，必須在各式各樣後端服務前呈現 API 的閘道也越來越複雜。 「Azure API 管理」已設計成讓您花費最少的功夫，即可處理具有路由規則、存取控制、速率限制、監視、事件記錄及回應快取功能的複雜 API。 「Azure API 管理」支援 Service Fabric 服務探索、分割區解析及複本選取，能夠以智能方式將要求直接路由傳送到 Service Fabric 中的後端服務，讓您無須撰寫自己的無狀態 API 閘道。 

在此案例中，仍然是透過 Web 服務來提供 Web UI，以及透過「Azure API 管理」來管理和路由傳送 HTTP API 呼叫，如下圖所示：

![Service Fabric 搭配 Azure API 管理拓撲概觀][sf-apim-web-app]

## <a name="application-scenarios"></a>應用程式案例

Service Fabric 中的服務可以是無狀態或具狀態服務，並且可使用下列三種配置之一來加以分割：單一、int-64 範圍及具名。 服務端點解析需要識別特定服務執行個體的特定分割區。 解析服務的端點時，必須同時指定服務執行個體名稱 (例如 `fabric:/myapp/myservice`) 和服務的特定分割區，但單一分割區的情況除外。

「Azure API 管理」可以與無狀態服務、具狀態服務及任何資料分割配置的任意組合搭配使用。

## <a name="send-traffic-to-a-stateless-service"></a>將流量傳送到無狀態服務

在最簡單的案例中，會將流量轉送到無狀態服務執行個體。 為了達成此目的，「API 管理」作業會包含一個具有 Service Fabric 後端的輸入處理原則，此後端會對應至 Service Fabric 後端中特定的無狀態服務執行個體。 系統會將傳送到該服務的要求傳送到該無狀態服務執行個體地隨機複本。

#### <a name="example"></a>範例
在以下案例中，Service Fabric 應用程式包含一個名為 `fabric:/app/fooservice` 的無狀態服務，此服務會公開內部 HTTP API。 此服務執行個體名稱為已知的名稱，可直接以硬式編碼編寫在「API 管理」輸入處理原則中。 

![Service Fabric 搭配 Azure API 管理拓撲概觀][sf-apim-static-stateless]

## <a name="send-traffic-to-a-stateful-service"></a>將流量傳送到具狀態服務

與無狀態服務案例類似，系統可以將流量轉送到具狀態服務執行個體。 在此情況下，「API 管理」作業會包含一個具有 Service Fabric 後端的輸入處理原則，此後端會將要求對應至特定「具狀態」服務執行個體的特定分割區。 與每個要求對應的分割區是使用來自傳入 HTTP 要求的一些輸入資料 (例如 URL 路徑中的值) 透過 Lambda 方法計算來得出的。 您可以將原則設定為只將要求傳送到主要複本，或針對讀取作業傳送到隨機複本。

#### <a name="example"></a>範例

在以下案例中，Service Fabric 應用程式包含一個名為 `fabric:/app/userservice` 的已分割具狀態服務，此服務會公開內部 HTTP API。 此服務執行個體名稱為已知的名稱，可直接以硬式編碼編寫在「API 管理」輸入處理原則中。  

此服務是以 Int64 資料分割配置來分割的，此配置具有兩個分割區及跨 `Int64.MinValue` 到 `Int64.MaxValue` 的索引鍵範圍。 後端原則會透過將 URL 要求路徑中提供的 `id` 值轉換成 64 位元整數，來計算出一個在該範圍內的分割區索引鍵，不過這裡可以使用任何演算法來計算分割區索引鍵。 

![Service Fabric 搭配 Azure API 管理拓撲概觀][sf-apim-static-stateful]

## <a name="send-traffic-to-multiple-stateless-services"></a>將流量傳送到多個無狀態服務

在較進階的案例中，您可以定義一個將要求對應至多個服務執行個體的「API 管理」作業。 在此情況下，每個作業都會包含一個原則，此原則會根據來自傳入 HTTP 要求的值 (例如 URL 路徑或查詢字串) 將要求對應至特定的服務執行個體，而在具狀態服務的案例中，則是會對應至服務執行個體內的分割區。 

為了達成此目的，「API 管理」作業會包含一個具有 Service Fabric 後端的輸入處理原則，此後端會根據從傳入 HTTP 要求擷取到的值，對應至 Service Fabric 後端中的無狀態服務執行個體。 系統會將對服務執行個體發出的要求傳送到服務執行個體的隨機複本。

#### <a name="example"></a>範例

在此範例中，會為應用程式的每個使用者都建立一個新的無狀態服務，其中會使用下列公式為這些服務執行個體動態產生名稱：
 
 - `fabric:/app/users/<username>`

 每個服務都具有唯一的名稱，但並無法事先得知這些名稱，因為是根據使用者或系統管理員的輸入來建立服務，所以無法以硬式編碼編寫在 APIM 原則或路由規則中。 取而代之的是，會在後端原則定義中，從 URL 要求路徑中提供的 `name` 值產生作為要求傳送目的地的服務名稱。 例如︰

  - 對 `/api/users/foo` 發出的要求會路由傳送到服務執行個體 `fabric:/app/users/foo`
  - 對 `/api/users/bar` 發出的要求會路由傳送到服務執行個體 `fabric:/app/users/bar`

![Service Fabric 搭配 Azure API 管理拓撲概觀][sf-apim-dynamic-stateless]

## <a name="send-traffic-to-multiple-stateful-services"></a>將流量傳送到多個具狀態服務

與無狀態服務範例類似，「API 管理」可以將要求對應至多個「具狀態」服務執行個體，在此情況下，您可能也需要為每個具狀態服務執行個體執行分割區解析。

為了達成此目的，「API 管理」作業會包含一個具有 Service Fabric 後端的輸入處理原則，此後端會根據從傳入 HTTP 要求擷取到的值，對應至 Service Fabric 後端中的具狀態服務執行個體。 除了將要求對應至特定服務執行個體之外，也可以將要求對應至服務執行個體內的特定分割區，以及視需要對應至分割區內的主要複本或隨機次要複本。

#### <a name="example"></a>範例

在此範例中，會為應用程式的每個使用者都建立一個新的具狀態服務，其中會使用下列公式為這些服務執行個體動態產生名稱：
 
 - `fabric:/app/users/<username>`

 每個服務都具有唯一的名稱，但並無法事先得知這些名稱，因為是根據使用者或系統管理員的輸入來建立服務，所以無法以硬式編碼編寫在 APIM 原則或路由規則中。 取而代之的是，會在後端原則定義中，從 URL 要求路徑中提供的 `name` 值產生作為要求傳送目的地的服務名稱。 例如︰

  - 對 `/api/users/foo` 發出的要求會路由傳送到服務執行個體 `fabric:/app/users/foo`
  - 對 `/api/users/bar` 發出的要求會路由傳送到服務執行個體 `fabric:/app/users/bar`

每個服務執行個體也是以 Int64 資料分割配置來分割的，此配置具有兩個分割區及跨 `Int64.MinValue` 到 `Int64.MaxValue` 的索引鍵範圍。 後端原則會透過將 URL 要求路徑中提供的 `id` 值轉換成 64 位元整數，來計算出一個在該範圍內的分割區索引鍵，不過這裡可以使用任何演算法來計算分割區索引鍵。 

![Service Fabric 搭配 Azure API 管理拓撲概觀][sf-apim-dynamic-stateful]

## <a name="next-steps"></a>後續步驟

依照[教學課程](service-fabric-tutorial-deploy-api-management.md)來設定您第一個搭配「API 管理」的 Service Fabric 叢集，並透過「API 管理」將要求傳送到您的服務。

<!-- links -->

<!-- pics -->
[sf-apim-web-app]: ./media/service-fabric-api-management-overview/sf-apim-web-app.png
[sf-web-app-stateless-gateway]: ./media/service-fabric-api-management-overview/sf-web-app-stateless-gateway.png
[sf-apim-static-stateless]: ./media/service-fabric-api-management-overview/sf-apim-static-stateless.png
[sf-apim-static-stateful]: ./media/service-fabric-api-management-overview/sf-apim-static-stateful.png
[sf-apim-dynamic-stateless]: ./media/service-fabric-api-management-overview/sf-apim-dynamic-stateless.png
[sf-apim-dynamic-stateful]: ./media/service-fabric-api-management-overview/sf-apim-dynamic-stateful.png