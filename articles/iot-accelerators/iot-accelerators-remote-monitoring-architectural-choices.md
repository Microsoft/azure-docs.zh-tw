---
title: 遠端監視解決方案架構選擇 - Azure | Microsoft Docs
description: 此文章說明在遠端監視中所做的架構及技術選擇
author: timlaverty
manager: camerons
ms.author: timlav
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 09/12/2018
ms.topic: conceptual
ms.openlocfilehash: 09c5981701ffdee5f2e5dba47cc98c91d5df7526
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45603900"
---
# <a name="remote-monitoring-architectural-choices"></a>遠端監視架構選擇

Azure IoT 遠端監視解決方案加速器是具備 MIT 授權的開放原始碼解決方案加速器，其會導入常見的 IoT 情節 (例如裝置連線能力、裝置管理及串流處理)，讓客戶可加速開發程序。  遠端監視解決方案會遵循在[這裡](https://aka.ms/iotrefarchitecture)發行的建議 Azure IoT 參考架構。  

此文章描述在適用於遠端監視解決方案的每個子系統中所做的架構及技術選擇，並討論要考量的替代方案。  請務必注意，在遠端監視解決方案中所做的技術選擇不是實作遠端監視 IoT 解決方案的唯一方式。  技術性實作是用於建置成功應用程式的基準，而且應進行修改以符合客戶解決方案實作需要的技能、體驗及垂直應用程式。

## <a name="architectural-choices"></a>架構選擇

### <a name="microservices-serverless-and-cloud-native"></a>微服務、無伺服器及雲端原生

我們針對 IoT 應用程式所建議的架構是以雲端原生、微服務且無伺服器為基礎。  IoT 應用程式的不同子系統應建置為能以獨立方式部署且能夠獨立調整的個別服務。  這些屬性能夠在更新個別子系統時進行更好的調整且更有彈性，並提供根據每個子系統來選擇適當技術的彈性。  微服務可以使用多種技術來實作。 例如，使用 Docker 之類的容器技術搭配 Azure Functions 之類的無伺服器技術，或在 Azure App Service 之類的 PaaS 服務中裝載微服務。

## <a name="core-subsystem-technology-choices"></a>核心子系統技術選擇

此節將詳細說明在遠端監視解決方案中針對每個核心子系統所做的技術選擇。

![核心圖表](./media/iot-accelerators-remote-monitoring-architectural-choices/subsystem.png) 

### <a name="cloud-gateway"></a>雲端閘道
Azure IoT 中樞可作為遠端監視解決方案雲端閘道。  IoT 中樞提供與裝置的安全雙向通訊。 您可以在[這裡](https://azure.microsoft.com/services/iot-hub/) \(英文\) 深入了解 IoT 中樞。 針對 IoT 裝置連線能力，會使用 .NET Core 和 Java IoT 中樞 SDK。  此 SDK 會提供包裝 IoT 中樞 REST API 的包裝函式，並處理重試之類的案例。

### <a name="stream-processing"></a>串流處理
針對串流處理，遠端監視解決方案會使用 Azure 串流分析進行複雜的規則處理。  若客戶想要更簡單的規則，我們也提供自訂微服務來支援簡單規則的處理，即使此設定不屬於既有部署的一部分也一樣。 參考架構建議使用 Azure Functions 進行簡單的規則處理，以及使用 Azure 串流分析 (ASA) 進行複雜的規則處理。  

### <a name="storage"></a>儲存體
針對儲存體，遠端監視解決方案加速器會同時使用 Azure 時間序列深入解析和 Azure Cosmos DB。 Azure 時間序列深入解析會儲存來自您的已連線裝置並經由 IoT 中樞傳輸的訊息。 解決方案加速器會針對其他儲存體使用 Azure Cosmos DB，例如冷儲存體、規則定義、警示和組態設定。 Azure Cosmos DB 是針對 IoT 應用程式所建議的一般用途暖儲存體解決方案，即使 Azure 時間序列深入解析和 Azure Data Lake 等解決方案適用於多個使用案例也一樣。 透過 Azure 時間序列深入解析，您可以識別趨勢和異常狀況來針對時間序列感應器資料獲得更深入的見解，這使您得以分析根本原因，並避免代價高昂的停機時間。 

> [!NOTE]
> Azure 中國雲端目前不提供時間序列深入解析。 Azure 中國雲端中的新遠端監視解決方案加速器部署將 Cosmos DB 使用於所有的儲存體。

### <a name="business-integration"></a>商務整合
遠端監視解決方案中的商務整合僅限於產生警示，其放置於暖儲存體中。 您可以透過整合解決方案和 Azure Logic Apps，來執行進一步的商務整合。

### <a name="user-interface"></a>使用者介面
Web UI 是利用 JavaScript React 所建置的。  React 提供業界常用的 Web UI 架構，而且類似於其他常用架構 (例如 Angular)。  

### <a name="runtime-and-orchestration"></a>執行階段與協調流程
針對遠端監視解決方案中的子系統實作所選擇的應用程式執行階段是具有 Kubernetes 的 Docker 容器，可作為協調器進行水平調整。  此架構允許針對每個子系統進行個別調整定義，但從安全性觀點來看，會產生 DevOps 成本以使 VM 和容器保持最新狀態。  Docker 和 Kubernetes 的替代方案包括在 PaaS 服務 (例如 Azure App Service) 中裝載微服務，或使用 Service Fabric、DCOS、Swarm 等作為協調器。

## <a name="next-steps"></a>後續步驟
* 在[此處](https://www.azureiotsolutions.com/)部署遠端監視解決方案。
* 在 [C#](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet/) \(英文\) 和[Java](https://github.com/Azure/azure-iot-pcs-remote-monitoring-java/) \(英文\) 中探索 GitHub 程式碼。  
* 在[這裡](https://aka.ms/iotrefarchitecture) \(英文\) 深入了解 IoT 參考架構。
