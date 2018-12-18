---
title: 使用 Azure Service Fabric Explorer 視覺化叢集 | Microsoft Docs
description: Service Fabric Explorer 是一種應用程式，可檢查和管理 Microsoft Azure Service Fabric 叢集中的雲端應用程式和節點。
services: service-fabric
documentationcenter: .net
author: mikkelhegn
manager: msfussell
editor: ''
ms.assetid: c875b993-b4eb-494b-94b5-e02f5eddbd6a
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/02/2018
ms.author: mikhegn
ms.openlocfilehash: 459dd86fd614cb185801b074cea70c36dc7f6ccb
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38972327"
---
# <a name="visualize-your-cluster-with-service-fabric-explorer"></a>使用 Service Fabric 總管視覺化叢集

Service Fabric Explorer (SFX) 是開放原始碼工具，可檢查及管理 Azure Service Fabric 叢集。 Service Fabric Explorer 是適用於 Windows、macOS 和 Linux 的桌面應用程式。

## <a name="service-fabric-explorer-download"></a>Service Fabric Explorer 下載

您可以使用下列連結將 Service Fabric Explorer 下載為桌面應用程式：

- Windows
  - https://aka.ms/sfx-windows

- Linux
  - https://aka.ms/sfx-linux-x86
  - https://aka.ms/sfx-linux-x64

- macOS
  - https://aka.ms/sfx-macos

> [!NOTE]
> Service Fabric Explorer 的桌面版本可能比叢集支援有更多或更少功能。 您可以切換回部署至叢集的 Service Fabric Explorer 版本，以確認完整功能的相容性。
>
>

### <a name="running-service-fabric-explorer-from-the-cluster"></a>從叢集執行 Service Fabric Explorer

Service Fabric Explorer 也裝載於 Service Fabric 叢集的 HTTP 管理端點。 若要在 Web 瀏覽器中啟動 SFX，您可以在任何瀏覽器中瀏覽至叢集的 HTTP 管理端點，例如 https://clusterFQDN:19080 。

針對開發人員工作站設定，您可以瀏覽至 https://localhost:19080/Explorer，在本機叢集上啟動 Service Fabric Explorer。 請參閱此文章以[準備您的開發環境](service-fabric-get-started.md)。

## <a name="connect-to-a-service-fabric-cluster"></a>連線至 Service Fabric 叢集
若要連線至 Service Fabric 叢集，您需要叢集管理端點 (FQDN/IP) 和 HTTP 管理端點連接埠 (預設為 19080)。 例如 https://mysfcluster.westus.cloudapp.azure.com:19080 。 使用 [連線至 localhost] 核取方塊，連線至工作站上的本機叢集。

### <a name="connect-to-a-secure-cluster"></a>連線到安全的叢集
您可以為使用憑證或使用 Azure Active Directory (AAD) 來控制用戶端對您 Service Fabric 叢集的存取。

如果您嘗試連線到安全叢集，則視叢集的設定而定，您將必須出示用戶端憑證或使用 AAD 來登入。

## <a name="video-tutorial"></a>影片教學課程

若要了解如何使用 Service Fabric Explorer，請觀賞下列 Microsoft Virtual Academy 影片︰

> [!NOTE]
> 這段影片會示範裝載於 Service Fabric 叢集的 Service Fabric Explorer，而非桌面版本。
>
>

[<center><img src="./media/service-fabric-visualizing-your-cluster/SfxVideo.png" WIDTH="360" HEIGHT="244"></center>](https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=bBTFg46yC_9806218965)

## <a name="understand-the-service-fabric-explorer-layout"></a>了解 Service Fabric 總管配置
您可以使用左邊的樹狀目錄來瀏覽 Service Fabric 總管。 在樹狀目錄的根目錄，叢集儀表板會提供您叢集的概觀，包括應用程式和節點健康情況的摘要。

![Service Fabric 總管叢集儀表板][sfx-cluster-dashboard]

### <a name="view-the-clusters-layout"></a>檢視叢集的配置
Service Fabric 叢集中的節點會橫跨容錯網域和升級網域的二維方格放置。 這個位置可確保您的應用程式在發生硬體故障及應用程式升級時仍然可用。 您可以使用叢集對應檢視目前叢集的配置方式。

![Service Fabric 總管叢集對應][sfx-cluster-map]

### <a name="view-applications-and-services"></a>檢視應用程式和服務
叢集包含兩個樹狀子目錄：一個用於應用程式，另一個用於節點。

您可以使用應用程式檢視瀏覽 Service Fabric 的邏輯階層：應用程式、服務、資料分割，以及複本。

在以下範例中，**MyApp** 應用程式是由 **MyStatefulService** 與 **WebService** 兩個服務組成。 由於 **MyStatefulService** 可設定狀態，因此它包含一個具有一個主要複本和兩個次要複本的資料分割。 對比之下，WebSvcService 則無狀態，而且只包含單一執行個體。

![Service Fabric 總管應用程式檢視][sfx-application-tree]

在樹狀目錄的每個層級，主要窗格會顯示項目的相關資訊。 例如，您可以看到特定服務的健康狀態和版本。

![Service Fabric 總管基本資訊窗格][sfx-service-essentials]

### <a name="view-the-clusters-nodes"></a>檢視叢集的節點
節點檢視會顯示叢集的實體配置。 對於指定的節點，您可以檢查已經在該節點上部署程式碼的應用程式。 更具體地說，您可以看到目前在那裡執行的複本。

## <a name="actions"></a>動作
「Service Fabric 總管」提供一個對您叢集內的節點、應用程式及服務快速叫用動作的方式。

例如，若要刪除某個應用程式執行個體，請從左邊的樹狀目錄選擇該應用程式，然後選擇 [動作] > [刪除應用程式]。

![在 Service Fabric 總管中刪除應用程式][sfx-delete-application]

> [!TIP]
> 您可以按下每個元素旁邊的省略符號來執行相同動作。
>
> 可以透過 Service Fabric 總管執行的每個動作也都可以透過 PowerShell 或 REST API 執行，以實現自動化。
>
>

您也可以使用 Service Fabric Explorer，為指定的應用程式類型和版本建立應用程式執行個體。 在樹狀檢視中選擇應用程式類型，然後在右邊窗格中按一下您想要的版本旁邊的 [建立應用程式執行個體]  連結。

![在 Service Fabric Explorer 中建立應用程式執行個體][sfx-create-app-instance]

> [!NOTE]
> 建立應用程式執行個體時，Service Fabric Explorer 不支援參數。 應用程式執行個體會使用預設參數值。
>
>

## <a name="next-steps"></a>後續步驟
* [在 Visual Studio 中管理 Service Fabric 應用程式](service-fabric-manage-application-in-visual-studio.md)
* [使用 PowerShell 部署 Service Fabric 應用程式](service-fabric-deploy-remove-applications.md)

<!--Image references-->
[sfx-cluster-dashboard]: ./media/service-fabric-visualizing-your-cluster/SfxClusterDashboard.png
[sfx-cluster-map]: ./media/service-fabric-visualizing-your-cluster/SfxClusterMap.png
[sfx-application-tree]: ./media/service-fabric-visualizing-your-cluster/SfxApplicationTree.png
[sfx-service-essentials]: ./media/service-fabric-visualizing-your-cluster/SfxServiceEssentials.png
[sfx-delete-application]: ./media/service-fabric-visualizing-your-cluster/SfxDeleteApplication.png
[sfx-create-app-instance]: ./media/service-fabric-visualizing-your-cluster/SfxCreateAppInstance.png
