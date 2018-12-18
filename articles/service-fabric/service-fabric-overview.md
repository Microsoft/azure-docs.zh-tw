---
title: Azure 上的 Service Fabric 概觀 | Microsoft Docs
description: 組成多個微服務以提供調整和恢復功能的 Service Fabric 概觀。 Service Fabric 是一種分散式系統平台，用來建置可調整、可靠且輕鬆管理的雲端應用程式。
services: service-fabric
documentationcenter: .net
author: msfussell
manager: timlt
editor: masnider
ms.assetid: bbcc652a-a790-4bc4-926b-e8cd966587c0
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: overview
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/20/2017
ms.author: msfussell
ms.custom: mvc
ms.openlocfilehash: 5c7586ae2ff1a648348c1155a528035fa72c7744
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44299809"
---
# <a name="overview-of-azure-service-fabric"></a>Azure Service Fabric 概觀
Azure Service Fabric 是一個分散式系統平台，可讓您輕鬆封裝、部署及管理可調整和可信賴的微服務與容器。 Service Fabric 也解決了開發及管理雲端原生應用程式時所面臨的重大挑戰。 開發人員與管理員能夠避免複雜的基礎結構問題，專注於實作關鍵且嚴格要求之可調整、可信賴且可管理的工作負載。 Service Fabric 代表新一代的平台，可用於建置及管理這些在容器中執行的企業級、第一層雲端級別應用程式。

此短片將介紹 Service Fabric 和微服務：
> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-Service-Fabric/player]

## <a name="applications-composed-of-microservices"></a>由微服務組成的應用程式 
Service Fabric 可讓您建置及管理由微服務組成的應用程式，這類應用程式可調整且可靠，並以高密度方式在共用的機器集區 (稱為叢集) 上執行。 它提供精密、輕量的執行階段，可建置在容器中執行之分散式、可調整的無狀態和具狀態微服務。 此外，還提供完整的應用程式管理功能，可佈建、部署、監視、升級/修補及刪除已部署的應用程式，包括容器化服務。

Service Fabric 提供技術支援給現今許多 Microsoft 服務，包括 Azure SQL Database、Azure Cosmos DB、Cortana、Microsoft Power BI、Microsoft Intune、「Azure 事件中樞」、「Azure IoT 中樞」、Dynamics 365、「商務用 Skype」以及許多核心 Azure 服務。

Service Fabric 是針對建立雲端原生服務量身打造而成，此類服務可視需要以小規模開始，接著擴充為包含成千上萬個機器的大規模服務。

現今的網際網路級別服務是使用微服務建立。 微服務範例包括通訊協定閘道器、使用者設定檔、購物車、清查處理、佇列和快取。 Service Fabric 是一個微服務平台，為可以是無狀態或具狀態的每個微服務 (或容器) 都提供一個唯一的名稱。

Service Fabric 為由微服務組成的應用程式，提供完整的執行階段與生命週期管理功能。 其將微服務裝載在容器中，這些容器部署在整個 Service Fabric 叢集之中並已啟動。 從虛擬機器改成用容器，可讓密度極為大幅地增加。 同樣地，當您從容器移至這些容器中的微服務時，也可讓密度再提升一個量級。 例如，單一 Azure SQL Database 叢集是由數百個機器所組成，這些機器執行數萬個容器，而容器總共裝載了數十萬個資料庫。 每個資料庫都是 Service Fabric 的可設定狀態微服務。 

如需有關微服務方法的詳細資訊，請閱讀 [為何要用微服務方式建置應用程式？](service-fabric-overview-microservices.md)

## <a name="container-deployment-and-orchestration"></a>容器部署和協調流程
Service Fabric 是將微服務部署至整個機器叢集的 Microsoft [容器協調者](service-fabric-cluster-resource-manager-introduction.md)。 微服務的開發方式有許多種，從使用 [Service Fabric 程式設計模型](service-fabric-choose-framework.md)、[ASP.NET Core](service-fabric-reliable-services-communication-aspnetcore.md)，到部署[您選擇的任何程式碼](service-fabric-guest-executables-introduction.md)，應有盡有。 重要的是，您可以在相同應用程式中同時混合容器中的服務和處理序中的服務。 如果您只是想要[部署和管理容器](service-fabric-containers-overview.md)，則 Service Fabric 作為容器協調者是一個最佳選擇。

## <a name="any-os-any-cloud"></a>任何 OS、任何雲端
Service Fabric 可在任何環境執行。 您可以在許多環境 (包括 Azure 或內部部署、Windows Server 上或 Linux 上) 建立 Service Fabric 的叢集。 您甚至可以在其他公用雲端上建立叢集。 此外，SDK 中的開發環境與生產環境「完全相同」，不涉及任何模擬器。 換句話說，在本機開發叢集上執行的項目，也會部署到其他環境中的叢集。

![Service Fabric 平台][Image1]

對於 Windows 開發，Service Fabric.NET SDK 與 Visual Studio 和 Powershell 整合。 請參閱[在 Windows 上準備您的開發環境](service-fabric-get-started.md)。 對於 Linux 開發，Service Fabric Java SDK 與 Eclipse 整合，Yeoman 用於為 Java、.NET Core 和容器應用程式產生範本。 請參閱[在 Linux 上準備您的開發環境](service-fabric-get-started.md)

如需建立叢集的詳細資訊，請參閱[在 Windows Server 或 Linux 上建立叢集](service-fabric-deploy-anywhere.md)，或是[透過 Azure 入口網站](service-fabric-cluster-creation-via-portal.md)針對 Azure 建立叢集。

## <a name="stateless-and-stateful-microservices-for-service-fabric"></a>Service Fabric 的無狀態與具狀態微服務
Service Fabric 可讓您建置由微服務或容器組成的應用程式。 無狀態微服務 (如通訊協定閘道器、Web Proxy) 不會維護要求之外的可變動狀態及來自服務的回應。 Azure 雲端服務背景工作角色即為無狀態服務的範例。 可設定狀態的微服務 (如使用者帳戶、資料庫、裝置、購物車、佇列) 會維護要求及其回應外的可變動授權狀態。 現今的網際網路級別應用程式包含無狀態與可設定狀態微服務的組合。 

Service Fabric 的主要區別在於，它強烈著重在建置具狀態服務，不論是使用[內建的程式設計模型](service-fabric-choose-framework.md)，還是使用容器化具狀態服務來建置。 [應用程式案例](service-fabric-application-scenarios.md)說明使用具狀態服務的案例。


## <a name="application-lifecycle-management"></a>應用程式生命週期管理
Service Fabric 可支援雲端應用程式 (包括容器) 的完整應用程式生命週期和 CI/CD。 此生命週期包括開發到部署、每日管理、維護，到最終的解除委任。

Service Fabric 應用程式生命週期管理讓應用程式管理員和 IT 操作員能夠使用簡單、低接觸的工作流程來佈建、部署、修補與監視應用程式。 這些內建的工作流程能大幅降低 IT 操作員的負擔，讓應用程式持續可用。

大多數應用程式都是由無狀態和具狀態微服務、容器及其他一起部署之可執行檔的組合所構成。 Service Fabric 在應用程式擁有強式型別，因此能夠部署多個應用程式執行個體。 每個執行個體都能獨立管理與升級。 重要的是，Service Fabric 可以部署容器或任何可執行檔，並讓它們都變得可靠。 例如，Service Fabric 可以部署 .NET、ASP.NET Core、node.js、Windows 容器、Linux 容器、Java 虛擬機器、指令碼、Angular 或實際上任何構成您應用程式的項目。

Service Fabric 已與 CI/CD 工具 (例如 [Azure Pipelines](https://www.visualstudio.com/team-services/)、[Jenkins](https://jenkins.io/index.html) 及 [Octopus Deploy](https://octopus.com/)) 整合，並且可與任何其他常用的 CI/CD 工具搭配使用。

如需應用程式生命週期管理的詳細資訊，請參閱[應用程式生命週期](service-fabric-application-lifecycle.md)。 如需有關如何部署任何程式碼的詳細資訊，請參閱[部署可體可執行檔](service-fabric-deploy-existing-app.md)。

## <a name="key-capabilities"></a>主要功能
藉由使用 Service Fabric，您可以：

* 部署至 Azure，或部署至執行 Windows 或 Linux 的內部部署資料中心，且程式碼零變更。 只要撰寫一次，就能夠在任何地方部署至任何 Service Fabric 叢集。
* 使用 Service Fabric 程式設計模型、容器或任何程式碼，來開發由微服務所組成的可調整應用程式。
* 開發高度可靠的無狀態與可設定狀態微服務。 使用具狀態微服務來簡化應用程式的設計。 
* 使用新奇的 Reliable Actors 程式設計模型，來建立具有獨立性程式碼與狀態的雲端物件。
* 部署並協調包含 Windows 容器和 Linux 容器的容器。 Service Fabric 是一個資料感知、具狀態的容器協調者。
* 以每部機器上數百或數千個應用程式或容器的高密度方式，快速部署應用程式。
* 並列部署相同應用程式的不同版本，並獨立升級每個應用程式。
* 無須停機即可管理應用程式的生命週期，包括重大升級與非重大升級。
* 相應放大或相應縮小叢集中的節點數目。 隨著您調整節點，應用程式也會自動調整。
* 監視和診斷應用程式的健全狀況，並設定用來執行自動修復的原則。
* 監看資源平衡器協調整個叢集的應用程式重新分配。 Service Fabric 會從失敗中復原，並會根據可用資源將負載分佈最佳化。

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## <a name="next-steps"></a>後續步驟
* 其他資訊：
  * [為何要用微服務方式建置應用程式？](service-fabric-overview-microservices.md)
  * [術語概觀](service-fabric-technical-overview.md)
* 設定 [Windows 開發環境](service-fabric-get-started.md)  
* 設定 [Linux 開發環境](service-fabric-get-started-linux.md)
* 了解 [Service Fabric 支援選項](service-fabric-support.md)

[Image1]: media/service-fabric-overview/Service-Fabric-Overview.png
