---
title: 雲端服務與 Service Fabric 之間的差異 | Microsoft Docs
description: 這是將應用程式從雲端服務移轉到 Service Fabric 的概念性概觀。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: ''
ms.assetid: 0b87b1d3-88ad-4658-a465-9f05a3376dee
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/02/2017
ms.author: vturecek
ms.openlocfilehash: 35ab4a9bdd66bf3571e7f189191550f88e17cee2
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34206477"
---
# <a name="learn-about-the-differences-between-cloud-services-and-service-fabric-before-migrating-applications"></a>移轉應用程式之前，先了解「雲端服務」與 Service Fabric 之間的差異。
Microsoft Azure Service Fabric 為次世代雲端應用程式平台，適用於高度可調整且高度可靠的分散式應用程式。 其中導入了許多封裝、部署、更新及管理分散式雲端應用程式的新功能。 

這是將應用程式從雲端服務移轉到 Service Fabric 的入門簡介。 內容著重於雲端服務與 Service Fabric 之間的架構及設計差異。

## <a name="applications-and-infrastructure"></a>應用程式與基礎結構
雲端服務與 Service Fabric 最基本的差異在於 VM、工作負載及應用程式之間的關係。 這裡的工作負載定義為您所撰寫來執行特定工作或提供服務的程式碼。

* **雲端服務的重點是將應用程式部署為 VM。** 您的程式碼緊密地與 VM 執行個體 (如 Web 或背景工作角色) 結合。 在雲端服務中部署工作負載，即為部署一或多個執行該工作負載的 VM 執行個體。 應用程式與 VM 沒有區別，因此對於應用程式也沒有正式的定義。 應用程式可以視為雲端服務部署中的一組 Web 或背景工作角色，或是整個雲端服務部署。 在此範例中，應用程式會顯示為一組角色執行個體。

![雲端服務應用程式和拓撲][1]

* **Service Fabric 的重點是將應用程式部署到現有的 VM 或執行 Service Fabric 的 Windows 或 Linux 電腦。** 您撰寫的服務完全與背後的基礎服務分離 (由 Service Fabric 應用程式平台抽象化)，因此可以將應用程式部署在多個環境。 Service Fabric 中的工作負載稱為「服務」，而一或多個服務會在 Service Fabric 應用程式平台上執行且正式定義的應用程式中形成群組。 可將多個應用程式部署到單一的 Service Fabric 叢集。

![Service Fabric 應用程式和拓撲][2]

Service Fabric 本身是在 Windows 或 Linux 執行的應用程式平台層，而雲端服務是用來部署受 Azure 管理且連接工作負載的 VM。
Service Fabric 應用程式模型有許多優點：

* 部署快速。 建立 VM 執行個體可能非常耗時。 在 Service Fabric 中，只會部署一次 VM 以形成裝載 Service Fabric 應用程式平台的叢集。 此後，應用程式封裝可快速部署到該叢集。
* 高密度裝載。 在雲端服務中，一個背景工作角色 VM 裝載一個工作負載。 在 Service Fabric 中，應用程式與執行它們的 VM 分離，表示您可以將大量的應用程式部署到小量的 VM，這樣可以降低大量部署的成本。
* Service Fabric 平台可在任何具備 Windows Server 或 Linux 電腦的位置執行，不論為 Azure 或內部部署。 平台透過背後的基礎結構提供抽象層，因此您的應用程式可在不同環境執行。 
* 管理分散式應用程式。 Service Fabric 這個平台不僅能裝載分散式應用程式，也協助獨立於主控 VM 或電腦生命週期之外管理其生命週期。

## <a name="application-architecture"></a>應用程式架構
雲端服務應用程式的基礎結構通常包括數個外部服務相依性 (例如服務匯流排、Azure 資料表與 Blob 儲存體、SQL、Redis 等)，以管理應用程式的狀態及資料，並和在雲端服務部署中的 Web 及背景工作角色通訊。 完整雲端服務應用程式的範例如下：  

![雲端服務架構][9]

Service Fabric 應用程式也可以選擇在完整應用程式中使用同樣的外部服務。 使用這個 Cloud Services 基礎結構範例，從雲端服務移轉到 Service Fabric 最簡單的路徑是，只將雲端服務部署以 Service Fabric 應用程式取代，整體基礎結構則保持相同。 變更最少量的程式碼，即可將 Web 和背景工作角色移植到 Service Fabric 無狀態服務。

![簡單移轉後的 Service Fabric 架構][10]

在這個階段，系統應該如先前般運作。 善加利用 Service Fabric 的具狀態服務，若適用則外部狀態存放區可初始化為具狀態服務。 這比單純移轉 Web 和背景工作角色至 Service Fabric 無狀態服務更複雜，因為需要撰寫如先前外部服務為應用程式提供之相同功能性的自訂程式碼。 這樣做的優點包括︰ 

* 移除外部相依性 
* 統一部署、管理及升級模型。 

初始化這些服務的基礎結構之結果的範例如下：

![完整移轉後的 Service Fabric 架構][11]

## <a name="communication-and-workflow"></a>通訊與工作流程
大部分的雲端服務應用程式由一個以上的層組成。 同樣地，Service Fabric 應用程式由一個以上的服務組成 (通常是許多服務)。 二種常見的通訊模型為直接通訊及透過外部耐久性儲存體。

### <a name="direct-communication"></a>直接通訊
使用直接通訊，層之間可以透過各自所公開的端點直接通訊。 在無狀態的環境 (例如雲端服務) 中，這表示選取 VM 角色的執行個體 (不論是隨機或循環資源配置以平衡負載)，並直接連接到其端點。

![雲端服務直接通訊][5]

 直接通訊是 Service Fabric 中常見的通訊模型。 Service Fabric 和雲端服務的關鍵差異在於，在雲端服務中您是連接到 VM，而在 Service Fabric 中是連接到服務。 這是重要差異，原因如下：

* Service Fabric 中的服務不受限於裝載它們的 VM。服務可以在叢集中移動，且已預期會因為幾個原因而移動：資源平衡、容錯移轉、應用程式和基礎結構更新，以及位置或負載的限制。 這表示服務執行個體的位址可隨時變更。 
* Service Fabric 中的 VM 可以託管多個服務，且每個有獨特的端點。

Service Fabric 提供服務探索機制 (稱為「名稱服務」)，可用來解析服務的端點位置。 

![Service Fabric 直接通訊][6]

### <a name="queues"></a>佇列
層與無狀態環境 (如雲端服務) 之間的常見通訊機制是使用外部儲存體佇列持久地儲存各層的工作狀態。 常見的案例是 Web 層將作業傳送到背景工作角色執行個體可以清除佇列並處理作業的 Azure Queue 或服務匯流排。

![雲端服務佇列通訊][7]

同樣的通訊模型可以用在 Service Fabric。 這有助於將現有的雲端服務應用程式移轉到 Service Fabric。 

![Service Fabric 直接通訊][8]

## <a name="next-steps"></a>後續步驟
從雲端服務移轉到 Service Fabric 最簡單的路徑是，只將雲端服務部署以 Service Fabric 應用程式取代，應用程式的整體基礎結構則大致上相同。 下列文章提供指南，以協助將 Web 和背景工作角色移轉到 Service Fabric 無狀態服務。

* [簡單移轉：將 Web 或背景工作角色轉換到 Service Fabric 無狀態服務](service-fabric-cloud-services-migration-worker-role-stateless-service.md)

<!--Image references-->
[1]: ./media/service-fabric-cloud-services-migration-differences/topology-cloud-services.png
[2]: ./media/service-fabric-cloud-services-migration-differences/topology-service-fabric.png
[5]: ./media/service-fabric-cloud-services-migration-differences/cloud-service-communication-direct.png
[6]: ./media/service-fabric-cloud-services-migration-differences/service-fabric-communication-direct.png
[7]: ./media/service-fabric-cloud-services-migration-differences/cloud-service-communication-queues.png
[8]: ./media/service-fabric-cloud-services-migration-differences/service-fabric-communication-queues.png
[9]: ./media/service-fabric-cloud-services-migration-differences/cloud-services-architecture.png
[10]: ./media/service-fabric-cloud-services-migration-differences/service-fabric-architecture-simple.png
[11]: ./media/service-fabric-cloud-services-migration-differences/service-fabric-architecture-full.png
