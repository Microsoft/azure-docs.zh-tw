---
title: 深入了解 Azure Service Fabric | Microsoft Docs
description: 了解 Azure Service Fabric 的核心概念和主要領域。 提供 Service Fabric 及如何建立微服務的延伸概觀。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 12/08/2017
ms.author: ryanwi
ms.openlocfilehash: d4b27feab5c1bb5913d2ba26f7f43aca9a899aa0
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43697664"
---
# <a name="so-you-want-to-learn-about-service-fabric"></a>您想要了解 Service Fabric 嗎？
Azure Service Fabric 是分散式系統平台，可讓您輕鬆封裝、部署及管理可調整和可信賴的微服務。  Service Fabric 有相當大的介面區，不過，要了解的方面很多。  本文提供 Service Fabric 的概述，並描述核心概念、程式設計模型、應用程式生命週期、測試、叢集及健康情況監視。 如需相關簡介及了解如何使用 Service Fabric 來建立微服務，請參閱[概觀](service-fabric-overview.md)和[什麼是微服務？](service-fabric-overview-microservices.md)。 本文並未包含完整的內容清單，但有連結到 Service Fabric 每個領域的概觀與入門文章。 

## <a name="core-concepts"></a>核心概念
[Service Fabric 術語](service-fabric-technical-overview.md)、[應用程式模型](service-fabric-application-model.md)及[支援的程式設計模型](service-fabric-choose-framework.md)提供更多概念和描述，但這裡提供基本概念。

<table><tr><th>核心概念</th><th>設計階段</th><th>執行階段</th></tr>
<tr><td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=tbuZM46yC_5206218965">
<img src="./media/service-fabric-content-roadmap/CoreConceptsVid.png" WIDTH="240" HEIGHT="162"></a></td>
<td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=tlkI046yC_2906218965"><img src="./media/service-fabric-content-roadmap/RunTimeVid.png" WIDTH="240" HEIGHT="162"></a></td>
<td><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=x7CVH56yC_1406218965">
<img src="./media/service-fabric-content-roadmap/RunTimeVid.png" WIDTH="240" HEIGHT="162"></a></td></tr>
</table>

### <a name="design-time-application-type-service-type-application-package-and-manifest-service-package-and-manifest"></a>設計階段：應用程式類型、服務類型、應用程式套件和資訊清單、服務套件和資訊清單
應用程式類型是指派給服務類型集合的名稱/版本。 這是在 *ApplicationManifest.xml* 檔案中定義，此檔案內嵌在應用程式套件目錄中。 然後，系統會將應用程式套件複製到 Service Fabric 叢集的映像存放區。 您可以接著從這個應用程式類型建立一個具名應用程式，然後在叢集內執行該應用程式。 

服務類型是指派給服務的程式碼封裝、資料封裝及組態封裝的名稱/版本。 這是在 ServiceManifest.xml 檔案中定義，此檔案內嵌在服務套件目錄中。 然後，應用程式套件的 *ApplicationManifest.xml* 檔案會參考此服務套件目錄。 在叢集內，建立具名應用程式之後，可以從應用程式類型的其中一個服務類型建立具名服務。 服務類型會由其 *ServiceManifest.xml* 檔案來描述。 服務類型包含在執行階段載入的可執行程式碼服務組態設定，以及服務所取用的靜態資料。

![Service Fabric 應用程式類型和服務類型][cluster-imagestore-apptypes]

應用程式套件是一個磁碟目錄，其中包含應用程式類型的 *ApplicationManifest.xml* 檔案，此檔案會參考組成應用程式類型之每個服務類型的服務套件。 例如，電子郵件應用程式類型的應用程式封裝可能包含指向佇列服務封裝、前端服務封裝、資料庫服務封裝的參考。 應用程式封裝目錄中的檔案會複製到 Service Fabric 叢集的映像存放區。 

服務套件是一個磁碟目錄，其中包含服務類型的 *ServiceManifest.xml* 檔案，此檔案會參考服務類型的程式碼、靜態資料及組態套件。 應用程式類型的 *ApplicationManifest.xml* 檔案會參考服務套件目錄中的檔案。 例如，服務套件可能會參考構成資料庫服務的程式碼、靜態資料和組態封裝。

### <a name="run-time-clusters-and-nodes-named-applications-named-services-partitions-and-replicas"></a>執行階段︰叢集和節點、具名應用程式、具名服務、分割區及複本
[Service Fabric 叢集](service-fabric-deploy-anywhere.md)是一組由網路連接的虛擬或實體機器，可用來將您的微服務部署到其中並進行管理。 叢集可擴充至數千部機器。

隸屬於叢集的機器或 VM 即稱為節點。 需為每個節點指派節點名稱 (字串)。 節點具有各種特性，如 placement 屬性。 每個電腦或 VM 皆有自動啟動的 Windows 服務 `FabricHost.exe`，該服務會在開機時開始執行，然後啟動兩個執行檔：`Fabric.exe` 和 `FabricGateway.exe`。 這兩個執行檔構成節點。 針對開發或測試案例，您可以藉由執行多個 `Fabric.exe` 和 `FabricGateway.exe` 執行個體，在單一電腦或 VM 上裝載多個節點。

具名應用程式是執行一或多個特定功能之具名服務的集合。 服務會執行完整且獨立的函數 (其可獨立於其他服務啟動和執行)，並且是由程式碼、組態和資料組成。 在應用程式封裝被複製到映像存放區之後，您可藉由指定應用程式封裝的應用程式類型 (使用其名稱/版本)，在叢集內建立應用程式的執行個體。 系統會為每個應用程式類型執行個體指派一個看起來如下的 URI 名稱：*fabric:/MyNamedApp*。 在叢集中，您可以從單一應用程式類型建立多個具名應用程式。 也可以從不同的應用程式類型建立具名應用程式。 每個具名應用程式都是獨立管理和控制版本。

在建立具名應用程式之後，您可藉由指定服務類型 (使用其名稱/版本)，在叢集內建立它其中一種服務類型的執行個體。 需為每個服務類型執行個體指派一個 URI (名稱範圍需在其具名應用程式的 URI 之下)。 例如，如果您在 "MyNamedApp" 具名應用程式內建立 "MyDatabase" 具名服務，URI 看起來會像這樣：*fabric:/MyNamedApp/MyDatabase*。 您可以在具名應用程式內建立一或多個具名服務。 每個具名服務可以有自己的分割配置和執行個體/複本計數。 

服務類型有兩種：無狀態和具狀態。 無狀態服務不會在服務內儲存狀態。 無狀態服務完全沒有持續性儲存體或可以將持續狀態儲存外部儲存體服務中，例如 Azure 儲存體、Azure SQL Database 或 Azure Cosmos DB。 具狀態服務會將狀態儲存在服務內並使用 Reliable Collections 或 Reliable Actors 程式設計模型來管理狀態。 

建立具名服務時，您需指定資料分割配置。 含有大量狀態的服務會跨資料分割切割其資料。 每個資料分割會負責服務完整狀態的一部分，其會分散至全體叢集的節點。  

下圖顯示應用程式和服務執行個體、分割和複本之間的關聯性。

![服務內的分割和複本][cluster-application-instances]

### <a name="partitioning-scaling-and-availability"></a>資料分割、調整大小和可用性
[資料分割](service-fabric-concepts-partitioning.md)並非 Service Fabric 所獨有。 資料分割是一種知名的資料分割形式，也稱為分區化。 含有大量狀態的具狀態服務，會跨資料分割切割其資料。 每個資料分割會負責服務完整狀態的一部分。 

每個資料分割的複本會分散至各個叢集節點，以允許[調整](service-fabric-concepts-scalability.md)具名服務的狀態規模。 資料需求成長時，資料分割也會成長，而 Service Fabric 會重新平衡全體節點之間的資料分割，以期有效率地使用硬體資源。 若您新增節點至叢集，則 Service Fabric 會重新平衡全體增加節點數的資料分割複本。 整體應用程式效能會有所改善，改善，並減少爭用記憶體的存取權。 若未有效率地使用叢集中的節點，您可減少叢集中的節點數目。 Service Fabric 會再次重新平衡全體減少節點數的資料分割複本，以善加使用每個節點上的硬體。

在分割內，無狀態的具名服務會有執行個體，而具狀態的具名服務則有複本。 通常，無狀態具名服務只會有 1 個分割，因為它們有沒有內部狀態。 資料分割執行個體提供[可用性](service-fabric-availability-services.md)。 若某個執行個體失敗，其他執行個體會繼續正常運作，接著 Service Fabric 會建立新的執行個體。 具狀態的具名服務會在複本中維持其狀態，且每個資料分割都有自己的複本集。 讀取與寫入作業會在單一複本上執行 (稱為「主要複本」)。 從寫入作業對狀態進行的變更，會複寫至多個其他複本 (稱為「作用中次要複本」)。 複本失敗失敗時，Service Fabric 會從現有複本建立新的複本。

## <a name="stateless-and-stateful-microservices-for-service-fabric"></a>Service Fabric 的無狀態與具狀態微服務
Service Fabric 可讓您建置由微服務或容器組成的應用程式。 無狀態微服務 (如通訊協定閘道器、Web Proxy) 不會維護要求之外的可變動狀態及來自服務的回應。 Azure 雲端服務背景工作角色即為無狀態服務的範例。 可設定狀態的微服務 (如使用者帳戶、資料庫、裝置、購物車、佇列) 會維護要求及其回應外的可變動授權狀態。 現今的網際網路級別應用程式包含無狀態與可設定狀態微服務的組合。 

Service Fabric 的主要區別在於，它強烈著重在建置具狀態服務，不論是使用[內建的程式設計模型](service-fabric-choose-framework.md)，還是使用容器化具狀態服務來建置。 [應用程式案例](service-fabric-application-scenarios.md)說明使用具狀態服務的案例。

為什麼有具狀態的微服務以及無狀態的微服務？ 兩個主要原因如下：

* 藉由將程式碼和資料放在相同電腦上且盡量靠近，您可以建立高輸送量、低延遲、容錯的線上交易處理 (OLTP) 服務。 其中的一些範例包括互動式店面、搜尋、物聯網 (IoT) 系統、交易系統、信用卡處理和詐欺偵測系統，以及個人記錄管理。
* 您可以簡化應用程式的設計。 可設定狀態的微服務不需要其他佇列與快取，但傳統上為滿足純粹無狀態應用程式的可用性與延遲需求則需要這些項目。 可設定狀態服務有高可用性、低延遲性的特性，整體來說可減少需要在應用程式中管理的移動組件。

## <a name="supported-programming-models"></a>支援的程式設計模型
Service Fabric 提供多種撰寫和管理服務的方式。 服務可以使用 Service Fabric API 來充分利用平台的功能和應用程式架構。 服務也可以是以任何語言撰寫並裝載於 Service Fabric 叢集上的已編譯可執行程式。 如需詳細資訊，請參閱[支援的程式設計模型](service-fabric-choose-framework.md)。

### <a name="containers"></a>容器
根據預設，Service Fabric 會以處理程序形式部署和啟用這些服務。 Service Fabric 也可以在[容器](service-fabric-containers-overview.md)中部署服務。 重要的是，您可以在相同應用程式中混合容器中的服務和處理序中的服務。 Service Fabric 支援在 Windows Server 2016 上部署 Linux 容器和 Windows 容器。 您可以在容器中部署現有的應用程式、無狀態服務或具狀態服務。 

### <a name="reliable-services"></a>Reliable Services
[Reliable Services](service-fabric-reliable-services-introduction.md) 是輕量型的服務撰寫架構，這些服務可與 Service Fabric 平台整合，而從一組完整的平台功能受益。 Reliable Services 可以是無狀態的 (類似大多數服務平台，例如「Azure 雲端服務」中的 Web 伺服器或「背景工作角色」)，其中狀態會保存在外部解決方案中，例如 Azure DB 或「Azure 資料表儲存體」。 Reliable Services 也可以是具狀態的，其狀態是使用 Reliable Collections 直接保存在服務本身中。 狀態透過複寫變得[高度可用](service-fabric-availability-services.md)，並透過[資料分割](service-fabric-concepts-partitioning.md)散發，全由 Service Fabric 自動管理。

### <a name="reliable-actors"></a>Reliable Actors
[Reliable Actor](service-fabric-reliable-actors-introduction.md) 架構是建置在 Reliable Services 上的應用程式架構，它會根據動作項目設計模式來實作 Virtual Actor 模式。 Reliable Actor 架構使用獨立的計算單位，和以單一執行緒方式執行稱為動作項目的狀態。 Reliable Actor 架構提供動作項目的內建通訊，以及預先設定狀態持續性和相應放大組態。

### <a name="aspnet-core"></a>ASP.NET Core
Service Fabric 與 [ASP.NET Core](service-fabric-reliable-services-communication-aspnetcore.md) 整合，可作為建置 Web 和 API 應用程式的最高等級程式設計模型。  可在 Service Fabric 中以兩種方式使用 ASP.NET Core︰

- 裝載作為客體可執行檔。 這主要用於在 Service Fabric 上執行現有的 ASP.NET Core 應用程式而無須變更程式碼。
- 在 Reliable Service 內部執行。 這可與 Service Fabric 執行階段更緊密整合，並可允許具狀態 ASP.NET Core 服務。

### <a name="guest-executables"></a>客體可執行檔
[客體可執行檔](service-fabric-guest-executables-introduction.md)是以任何語言所撰寫，且與其他服務一同裝載於 Service Fabric 叢集的現有任意可執行檔。 客體可執行檔未直接與 Service Fabric API 整合。 不過，它們仍然受惠於功能和平台供應項目，例如自訂健康情況和負載報告，以及透過呼叫 REST API 來探索服務。 它們也具備完整的應用程式生命週期支援。 

## <a name="application-lifecycle"></a>應用程式生命週期
如同其他平台，Service Fabric 上的應用程式通常會經歷下列階段：設計、開發、測試、部署、升級、維護和移除。 從開發到部署、到每日管理、維護，以及最終的解除委任，Service Fabric 為雲端應用程式的完整應用程式生命週期提供第一等的支援。 服務模型可以啟用數個不同的角色，在應用程式生命週期中獨立參與。 [Service Fabric 應用程式生命週期](service-fabric-application-lifecycle.md)說明 API 的概觀，以及不同的角色如何在 Service Fabric 應用程式生命週期的各個階段使用 API。 

您可使用 [PowerShell cmdlet](/powershell/module/ServiceFabric/)、[CLI 命令](service-fabric-sfctl.md)、[C# APIs](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient)、[Java APIs](/java/api/system.fabric) 和 [REST APIs](/rest/api/servicefabric/) 來管理整個應用程式生命週期。 您亦可使用 [Visual Studio Team Services](service-fabric-set-up-continuous-integration.md) 或 [Jenkins](service-fabric-cicd-your-linux-applications-with-jenkins.md) 等工具，設定持續整合/持續部署管線。

下列 Microsoft Virtual Academy 影片說明如何管理應用程式生命週期︰<center><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=My3Ka56yC_6106218965">
<img src="./media/service-fabric-content-roadmap/AppLifecycleVid.png" WIDTH="360" HEIGHT="244">
</a></center>

## <a name="test-applications-and-services"></a>測試應用程式與服務
為了建立真正雲端級別的服務，確保應用程式和服務可以禁得起真實世界失敗考驗便至關重要。 錯誤分析服務的設計用意，是測試建置於 Service Fabric 的服務。 您可以利用[錯誤分析服務](service-fabric-testability-overview.md)引發有意義的錯誤，針對應用程式執行完整的測試案例。 這些錯誤及案例會在受控制、安全且一致的情況下，執行及驗證服務在其生命週期會發生的各種狀態和轉換情形。

[動作](service-fabric-testability-actions.md)可鎖定服務並以個別錯誤執行測試。 服務開發人員可以使用這些動作當做建置組塊，來撰寫複雜的案例。 模擬錯誤的範例如下︰

* 重新啟動節點，以模擬任意次數的機器或 VM 重新開機情況。
* 移動您的具狀態服務複本，以模擬負載平衡、容錯移轉或應用程式升級。
* 在具狀態服務上叫用仲裁遺失，以建立無法繼續進行寫入作業的狀況，因為可接受新資料的「備份」或「次要」複本不足。
* 在具狀態服務上叫用資料遺失，以建立所有記憶體內狀態為完全抹除的狀況。

[案例](service-fabric-testability-scenarios.md)是由一或多個動作組成的複雜作業。 錯誤分析服務提供兩個內建的完整案例︰

* [混亂案例](service-fabric-controlled-chaos.md)會以很長的時間在整個叢集上模擬連續的交錯錯誤 (包括非失誤性和失誤性錯誤)。
* [容錯移轉測試案例](service-fabric-testability-scenarios.md#failover-test)是以特定服務資料分割為目標的混亂測試案例版本，且其他服務不會受到任何影響。

## <a name="clusters"></a>叢集
[Service Fabric 叢集](service-fabric-deploy-anywhere.md)是一組由網路連接的虛擬或實體機器，可用來將您的微服務部署到其中並進行管理。 叢集可擴充至數千部機器。 隸屬於叢集的機器或 VM 稱為叢集模式。 需為每個節點指派節點名稱 (字串)。 節點具有各種特性，如 placement 屬性。 每部電腦或 VM 皆有自動啟動服務 `FabricHost.exe`，此服務會在開機時開始執行，然後啟動兩個可執行檔：Fabric.exe 和 FabricGateway.exe。 這兩個執行檔構成節點。 在測試案例中，您可以藉由執行 `Fabric.exe` 和 `FabricGateway.exe` 的多個執行個體，在單一電腦或 VM 上裝載多個節點。

您可在執行 Windows Server 或 Linux 的虛擬機器或實體機器上，建立 Service Fabric 叢集。 只要有一組互連式 Windows Server 或 Linux 電腦，不論是在內部部署、Microsoft Azure 或透過任何雲端提供者，您皆可在任何環境中部署和執行 Service Fabric 應用程式。

下列 Microsoft Virtual Academy 影片說明 Service Fabric 叢集相關資訊：<center><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=tbuZM46yC_5206218965">
<img src="./media/service-fabric-content-roadmap/ClusterOverview.png" WIDTH="360" HEIGHT="244">
</a></center>

### <a name="clusters-on-azure"></a>Azure 上的叢集
我們可藉由在 Azure 上執行 Service Fabric 叢集，與其他的 Azure 功能和服務整合，而能夠更加輕鬆可靠地操作及管理叢集。 叢集為 Azure Resource Manager 資源，因此您可如同 Azure 中的其他任何資源般建立叢集模型。 Resource Manager 亦可輕鬆管理由叢集以單一單位方式使用的所有資源。 Azure 上的叢集會與 Azure 診斷及 Log Analytics 整合。 叢集節點類型是[虛擬機器擴展集](/azure/virtual-machine-scale-sets/index)，因此內建自動調整功能。

您可透過 [Azure 入口網站](service-fabric-cluster-creation-via-portal.md)、[範本](service-fabric-cluster-creation-via-arm.md)或 [Visual Studio](service-fabric-cluster-creation-via-visual-studio.md)，在 Azure 上建立叢集。

Linux 上的 Service Fabric 可讓您如同在 Windows 上一樣，在 Linux 上建置、部署和管理可用性高、可大幅調整的應用程式。 Service Fabric 架構 (Reliable Services 和 Reliable Actors) 除了可在 C# (.NET Core) 中使用，也可在 Linux 上的 Java 中使用。 您也可以使用任何語言或架構來建置 [來賓可執行檔服務](service-fabric-guest-executables-introduction.md) 。 也支援協調 Docker 容器。 Docker 容器可以執行使用 Service Fabric 架構的來賓可執行檔或原生 Service Fabric 服務。 如需詳細資訊，請參閱 [Linux 上的 Service Fabric](service-fabric-deploy-anywhere.md)。

有一些 Windows 上支援的功能，在 Linux 上未提供。 若要深入了解詳細資訊，請參閱 [Linux 與 Windows 上的 Service Fabric 差異](service-fabric-linux-windows-differences.md)。

### <a name="standalone-clusters"></a>獨立叢集
Service Fabric 提供安裝套件，可讓您在內部部署環境或任何雲端提供者上建立獨立 Service Fabric 叢集。 獨立叢集可讓您在任何想要的位置自由裝載叢集。 若您的資料受限於合規性或法規限制，或是您想要將資料保留在本機，則可以自行裝載叢集和應用程式。 Service Fabric 應用程式無須進行任何變更，即可在多個主控環境中執行，因此您的應用程式建置知識可以運用在不同的主控環境。 

[建立您的第一個 Service Fabric 獨立叢集](service-fabric-cluster-creation-for-windows-server.md)

尚不支援 Linux 獨立叢集。

### <a name="cluster-security"></a>叢集安全性
叢集必須受到安全保護，以防止未經授權使用者連線您的叢集，特別是當叢集上有生產工作負載正在執行時。 雖然可以建立不安全的叢集，但這樣做會在叢集向公用網際網路公開管理端點時，允許匿名使用者連線叢集。 無法在稍後為不安全的叢集啟用安全性：建立叢集時即會啟用叢集安全性。

叢集安全性案例包括：
* 節點對節點安全性
* 用戶端對節點安全性
* 角色型存取控制 (RBAC)

如需詳細資訊，請參閱[保護叢集](service-fabric-cluster-security.md)。

### <a name="scaling"></a>調整大小
若您新增節點至叢集，則 Service Fabric 會重新平衡全體增加節點數的資料分割複本和執行個體。 整體應用程式效能會有所改善，改善，並減少爭用記憶體的存取權。 若未有效率地使用叢集中的節點，您可減少叢集中的節點數目。 Service Fabric 會再次重新平衡全體減少節點數的資料分割複本和執行個體，以善加使用每個節點上的硬體。 您可透過[手動](service-fabric-cluster-scale-up-down.md)或[程式設計方式](service-fabric-cluster-programmatic-scaling.md)，調整 Azure 上叢集的規模。 您可[手動](service-fabric-cluster-windows-server-add-remove-nodes.md)調整獨立叢集的規模。

### <a name="cluster-upgrades"></a>叢集升級
新版本的 Service Fabric 執行階段會定期發行。 在叢集上執行執行階段或網狀架構升級，以確保一律執行[支援的版本](service-fabric-support.md)。 除了網狀架構升級，您亦可更新憑證或應用程式連接埠等叢集組態。

Service Fabric 叢集是由您所擁有，但由 Microsoft 部分管理的資源。 Microsoft 負責修補基礎 OS，以及在叢集上執行網狀架構升級。 您可以設定您的叢集 (當 Microsoft 發行新版本時) 接收自動網狀架構升級，或選擇選取您需要的受支援網狀架構版本。 您可透過 Azure 入口網站或 Resource Manager，設定網狀架構和組態升級。 如需詳細資訊，請參閱[升級 Service Fabric 叢集](service-fabric-cluster-upgrade.md)。 

獨立叢集是由您完全擁有的資源。 由您負責修補基礎 OS 和起始網狀架構升級。 若您的叢集可連線至 [https://www.microsoft.com/download](https://www.microsoft.com/download)，則您可將叢集設為自動下載和佈建新的 Service Fabric 執行階段套件。 接著您即會起始升級。 若您的叢集無法存取 [https://www.microsoft.com/download](https://www.microsoft.com/download)，則您可透過連線至網際網路的電腦下載新執行階段套件，然後再起始升級。 如需詳細資訊，請參閱[升級獨立 Service Fabric 叢集](service-fabric-cluster-upgrade-windows-server.md)。

## <a name="health-monitoring"></a>健康狀況監視
Service Fabric 引入了[健康狀態模型](service-fabric-health-introduction.md)，主要用於在特定實體 (例如叢集節點和服務複本) 上標示狀況不良的叢集和應用程式條件。 健康狀態模型使用健康狀態報告程式 (系統元件及看門狗)。 目標為輕易迅速的診斷並修復問題。 服務撰寫者必須預先考量健康狀態以及如何[設計健康狀態報告](service-fabric-report-health.md#design-health-reporting)等相關層面。 任何可能會影響到健康狀態的條件都需加以回報，尤其是如果它有助標示出接近根目錄的問題。 一旦在生產階段大規模啟動並執行服務，健康狀態資訊將可有效減少偵錯和調查工作所需的時間和心力。

Service Fabric 報告程式可監控感興趣的已識別條件。 它們會依據其本機檢視回報這些條件。 [健康狀態存放區](service-fabric-health-introduction.md#health-store) 可彙總報告程式送出的所有健康狀態資料，以判斷實體的健康狀態是否為全域良好。 必須為豐富、彈性且容易使用的模型。 健康狀態報告的品質可決定叢集的健康狀態檢視準確度。 不正確顯示出健康不良的問題之誤報，會對升級或其他使用健康狀態資料的服務產生負面影響。 這類服務的範例包括修復服務和警示機制。 因此，需要針對報告加以考量，才能讓其以最佳的方式擷取感興趣的條件。

報告可從以下項目完成：
* 受監視的 Service Fabric 服務複本或執行個體。
* 內部監視程式會部署為 Service Fabric 服務 (例如，可監視條件和問題報告的 Service Fabric 無狀態服務)。 可以在所有節點部署監視程式，也可以與受監視的服務相關。
* 在 Service Fabric 節點上執行，但未以 Service Fabric 服務實作的內部看門狗監視程式。
* 從 Service Fabric 叢集外探查資源的外部看門狗監視程式 (例如，監視 Gomez 等服務)。

Service Fabric 元件會針對叢集中的所有實體，提供現成的報告。 [系統健康狀態報告](service-fabric-understand-and-troubleshoot-with-system-health-reports.md)可讓您全盤掌握叢集和應用程式功能，並透過健康狀態標記問題。 系統健康狀態報告會針對應用程式和服務來確認實體是否已實作，並從 Service Fabric 執行階段的角度來確認其是行為是否正確。 報告並不會監控任何服務商務邏輯的健康狀態，也不會偵測是否有無回應的處理程序。 若要將特定的健康狀態資訊新增至服務的邏輯，請在服務中[實作自訂健康狀態報告](service-fabric-report-health.md)。

Service Fabric 提供多種[健康狀態報告檢視](service-fabric-view-entities-aggregated-health.md)方式，且會將此報告彙總於健康狀態資料存放區：
* [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) 或其他視覺效果工具。
* 健康情況查詢 (透過 [PowerShell](/powershell/module/ServiceFabric/)、[CLI](service-fabric-sfctl.md)、[C# FabricClient API](/dotnet/api/system.fabric.fabricclient.healthclient) 和 [Java FabricClient API](/java/api/system.fabric)，或是 [REST API](/rest/api/servicefabric))。
* 一般查詢會傳回一份實體清單，這些實體的其中一個屬性即為健康情況 (透過 Powershell、CLI、API 或 REST)。

下列 Microsoft Virtual Academy 影片說明 Service Fabric 健康狀態模型及其使用方式：<center><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=tevZw56yC_1906218965">
<img src="./media/service-fabric-content-roadmap/HealthIntroVid.png" WIDTH="360" HEIGHT="244">
</a></center>

## <a name="monitoring-and-diagnostics"></a>監視和診斷
不論任何環境，針對應用程式與服務的開發、測試及部署進行[監視和診斷](service-fabric-diagnostics-overview.md)，都極為重要。 如果您要規劃和實作監視和診斷功能，以協助確保應用程式和服務可在本機開發環境或生產環境中如預期般運作，Service Fabric 解決方案就是理想之選。

監視和診斷的主要目標是︰

- 偵測並診斷硬體和基礎結構的問題
- 偵測軟體和應用程式的問題，縮短服務中斷時間
- 了解資源耗用量，以及協助進行磁碟機作業決策
- 最佳化應用程式、服務及基礎結構效能
- 產生商業見解，並找出改善之處
 
監視與診斷的整體工作流程包含三個步驟：

1. 事件產生：這包含基礎結構 (叢集)、平台和應用程式/服務層級中的事件 (記錄、追蹤、自訂事件)
2. 事件彙總：產生的事件必須經過收集與彙總後，才能夠顯示
3. 分析︰事件必須經過視覺化並可藉由某種格式來存取，才能視需要進行分析及顯示

市面上很多產品已涵蓋這三個領域，因此您可以針對每個領域選擇不同的技術。 如需詳細資訊，請參閱[對 Azure Service Fabric 進行監視和診斷](service-fabric-diagnostics-overview.md)。

## <a name="next-steps"></a>後續步驟
* 了解如何[在 Azure 中建立叢集](service-fabric-cluster-creation-via-portal.md)或[在 Windows 上建立獨立叢集](service-fabric-cluster-creation-for-windows-server.md)。
* 嘗試使用 [Reliable Services](service-fabric-reliable-services-quick-start.md) 或 [Reliable Actors](service-fabric-reliable-actors-get-started.md) 程式設計模型來建立服務。
* 了解如何[從雲端服務移轉](service-fabric-cloud-services-migration-differences.md)。
* 了解如何[監視和診斷服務](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)。 
* 學習如何[測試您的應用程式和服務](service-fabric-testability-overview.md)。
* 學習如何[管理和組織叢集資源](service-fabric-cluster-resource-manager-introduction.md)。
* 查看 [Service Fabric 範例 ](http://aka.ms/servicefabricsamples)。
* 了解 [Service Fabric 支援選項](service-fabric-support.md)。
* 如需相關文章與公告資訊，請參閱[小組部落格](https://blogs.msdn.microsoft.com/azureservicefabric/)。


[cluster-application-instances]: media/service-fabric-content-roadmap/cluster-application-instances.png
[cluster-imagestore-apptypes]: ./media/service-fabric-content-roadmap/cluster-imagestore-apptypes.png
