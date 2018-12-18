---
title: Azure Service Fabric 模式和案例 | Microsoft Docs
description: 了解最佳作法，並證明可重複使用的模式來設計、開發及操作您的微服務，Service Fabric 上。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: d5aa75ff-98b9-4573-a2e5-7f5ab288157a
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 01/19/2018
ms.author: ryanwi
ms.openlocfilehash: 3bd77891cc7508eeb1fee2152d37478c654a7e37
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44294690"
---
# <a name="service-fabric-patterns-and-scenarios"></a>Service Fabric 模式和案例
如果您正在觀看如何使用 Azure Service Fabric 建置大規模微服務，請向負責設計及建置此平台即服務 (PaaS) 的專家學習。 從適當的架構開始，接著了解如何將您應用程式的資源最佳化。 [Service Fabric 模式和實務](https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=mudwqISGD_6005167344)課程會回答真實客戶最常詢問的 Service Fabric 案例和應用程式領域相關問題。
 
了解如何使用最佳作法和經過實證且可重複使用的模式，在 Service Fabric 上設計、開發及操作您的微服務。 取得 Service Fabric 的概觀，然後深入了解涵蓋叢集最佳化和安全性、移轉舊版應用程式、大規模 IoT、裝載遊戲引擎等的主題。 查看各種工作負載的連續傳遞，甚至取得 Linux 支援和容器的詳細資料。 

## <a name="introduction"></a>簡介
瀏覽最佳作法，並了解為何選擇平台即服務 (PaaS) 而不是基礎結構即服務 (IaaS)。 取得下列已經實證之應用程式設計原則的詳細資料。

<table><tr><th>影片</th><th>PowerPoint 簡報</th></tr>
<tr><td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=N2KwbbSGD_6405167344">
<img src="./media/service-fabric-patterns-and-scenarios/intro.png" WIDTH="360" HEIGHT="244">
</a></td><td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=mudwqISGD_6005167344">Service Fabric 簡介</a></td></tr>
</table>

## <a name="cluster-planning-and-management"></a>叢集規劃與管理
觀看此處的 Azure Service Fabric，了解容量規劃、叢集最佳化和叢集安全性。

<table><tr><th>影片</th><th>PowerPoint 簡報</th></tr>
<tr><td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=cyDYZcSGD_2805167344">
<img src="./media/service-fabric-patterns-and-scenarios/cluster.png" WIDTH="360" HEIGHT="244">
</a></td><td> <a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=E5B3nJSGD_805167344">叢集規劃與管理</a></td></tr>
</table>

## <a name="hyper-scale-web"></a>超大規模 Web
檢閱超大規模 Web 的概念，包括可用性和可靠性、超規模以及狀態管理。

<table><tr><th>影片</th><th>PowerPoint 簡報</th></tr>
<tr><td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=NgldAdSGD_405167344">
<img src="./media/service-fabric-patterns-and-scenarios/hyperscaleweb.png" WIDTH="360" HEIGHT="244">
</a></td><td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=CPMLBLSGD_7705167344">超大規模 Web</a></td></tr>
</table>

## <a name="iot"></a>IoT
探索 Azure Service Fabric 環境中的物聯網 (IoT)，包括 Azure IoT 管線、多重租用和大規模 IoT。

<table><tr><th>影片</th><th>PowerPoint 簡報</th></tr>
<tr><td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=naFUVeSGD_1505167344">
<img src="./media/service-fabric-patterns-and-scenarios/iot.png" WIDTH="360" HEIGHT="244">
</a></td><td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=kfqFWMSGD_6205167344">IoT</a></td></tr>
</table>

## <a name="gaming"></a>玩遊戲
察看回合制遊戲、互動式遊戲，以及裝載現有的遊戲引擎。

<table><tr><th>影片</th><th>PowerPoint 簡報</th></tr>
<tr><td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=6wECzeSGD_3805167344">
<img src="./media/service-fabric-patterns-and-scenarios/gaming.png" WIDTH="360" HEIGHT="244">
</a></td><td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=kfqFWMSGD_6205167344">遊戲</a></td></tr>
</table>

## <a name="continuous-delivery"></a>持續傳遞
探索相關概念，包括使用 Azure Pipelines、建置/封裝/發佈工作流程、多重環境設定及服務套件/共用的連續整合/連續傳遞。

<table><tr><th>影片</th><th>PowerPoint 簡報</th></tr>
<tr><td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=78h5ofSGD_305167344">
<img src="./media/service-fabric-patterns-and-scenarios/cd.png" WIDTH="360" HEIGHT="244">
</a></td><td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=VlENvOSGD_105167344">持續傳遞</a></td></tr>
</table>

## <a name="migration"></a>移轉
除了舊版應用程式的移轉以外，還要了解如何從雲端服務移轉。

<table><tr><th>影片</th><th>PowerPoint 簡報</th></tr>
<tr><td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=hd1cMgSGD_5605167344">
<img src="./media/service-fabric-patterns-and-scenarios/migration.png" WIDTH="360" HEIGHT="244">
</a></td><td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=GQAq4QSGD_8305167344">移轉</a></td></tr>
</table>

## <a name="containers-and-linux-support"></a>容器和 Linux 支援
取得「為何使用容器？」問題的答案 了解 Windows 容器、Linux 支援和 Linux 容器協調流程的概況簡介。 此外，了解如何將 .NET Core 應用程式移轉至 Linux。

<table><tr><th>影片</th><th>PowerPoint 簡報</th></tr>
<tr><td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=V1ERJhSGD_305167344">
<img src="./media/service-fabric-patterns-and-scenarios/containers.png" WIDTH="360" HEIGHT="244">
</a></td><td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/service-fabric-patterns-and-practices-16925?l=mlYsZRSGD_2105167344">容器和 Linux 支援</a></td></tr>
</table>

## <a name="next-steps"></a>後續步驟
既然您已了解 Service Fabric 模式和案例，請深入了解如何[建立和管理叢集](service-fabric-deploy-anywhere.md)、[將雲端服務應用程式移轉至 Service Fabric](service-fabric-cloud-services-migration-worker-role-stateless-service.md)、[設定連續傳遞](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)以及[部署容器](service-fabric-containers-overview.md)。
