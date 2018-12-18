---
title: 了解 Azure Service Fabric 術語 | Microsoft Docs
description: Service Fabric 的術語概觀 討論重要術語概念和文件其餘部分中使用的詞彙。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: chackdan;subramar
ms.assetid: 3a970679-e19e-43b3-9be8-71773f307c57
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/26/2018
ms.author: ryanwi
ms.openlocfilehash: e3da081f9b327031d6d1e0afd2f2fb52383bf933
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34212063"
---
# <a name="service-fabric-terminology-overview"></a>Service Fabric 術語概觀
Azure Service Fabric 是分散式系統平台，可讓您輕鬆封裝、部署及管理可調整和可信賴的微服務。 本文詳細說明 Service Fabric 中所使用的術語，以利您了解文件中使用的詞彙。

本節所列的概念也會在下列 Microsoft Virtual Academy 影片中討論︰<a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=tbuZM46yC_5206218965">核心概念</a>、<a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=tlkI046yC_2906218965">設計階段概念</a>和<a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=x7CVH56yC_1406218965">執行階段概念</a>。

## <a name="infrastructure-concepts"></a>基礎結構概念
**叢集**：由虛擬或實體機器連結組成的網路，微服務可於其中部署和管理。  叢集可擴充至數千部機器。

**節點**：屬於叢集一部分的電腦或 VM 都稱為節點。 需為每個節點指派節點名稱 (字串)。 節點具有各種特性，如 placement 屬性。 每個電腦或 VM 皆有自動啟動的 Windows 服務 `FabricHost.exe`，該服務會在開機時開始執行，然後啟動兩個執行檔：`Fabric.exe` 和 `FabricGateway.exe`。 這兩個執行檔構成節點。 在測試案例中，您可以藉由執行 `Fabric.exe` 和 `FabricGateway.exe` 的多個執行個體，在單一電腦或 VM 上裝載多個節點。

## <a name="application-concepts"></a>應用程式概念
**應用程式類型**：指派給服務類型的集合名稱/版本。 它是在 `ApplicationManifest.xml` 檔案中定義，並內嵌在應用程式套件目錄中。 然後，系統會將目錄複製到 Service Fabric 叢集的映像存放區。 然後，您可以從這個應用程式類型，在叢集內建立具名的應用程式。

如需詳細資訊，請參閱[應用程式模型](service-fabric-application-model.md)一文。

**應用程式套件**：磁碟目錄，包含此應用程式類型的 `ApplicationManifest.xml` 檔案。 參考組成此應用程式類型的每個服務類型的服務封裝。 應用程式封裝目錄中的檔案會複製到 Service Fabric 叢集的映像存放區。 例如，電子郵件應用程式類型的應用程式套件可能包含指向佇列服務套件、前端服務套件、資料庫服務套件的參考。

**具名應用程式**：您將應用程式套件複製到映像存放區之後，您會在叢集內建立應用程式的執行個體。 當您指定應用程式套件的應用程式類型時，使用其名稱或版本建立執行個體。 每個應用程式類型的執行個體會被指派一個看起來像這樣的統一資源識別項 (URI) 名稱：`"fabric:/MyNamedApp"`。 在叢集中，您可以從單一應用程式類型建立多個具名應用程式。 也可以從不同的應用程式類型建立具名應用程式。 每個具名應用程式都是獨立管理和控制版本。      

**服務類型**：指派給服務的程式碼套件、資料套件、設定套件的名稱/版本。 服務類型是在 `ServiceManifest.xml` 檔案中定義，並內嵌在服務套件目錄中。 然後，應用程式套件的 `ApplicationManifest.xml` 檔案會參考此服務套件目錄。 在叢集內，建立具名應用程式之後，可以從應用程式類型的其中一個服務類型建立具名服務。 服務類型的 `ServiceManifest.xml` 檔描述該服務。

如需詳細資訊，請參閱[應用程式模型](service-fabric-application-model.md)一文。

服務分為兩種：

* **無狀態**：當服務的持續狀態儲存在外部儲存服務 (例如「Azure 儲存體」、Azure SQL Database 或 Azure Cosmos DB) 時，請使用無狀態服務。 當服務沒有持續性儲存體時，請使用無狀態服務。 以計算機服務為例，首先要傳遞值給服務，服務用這些值執行計算，然後傳回結果。
* **具狀態**：當您要讓 Service Fabric 透過其 Reliable Collections 或 Reliable Actors 程式設計模型管理您的服務狀態，則使用具狀態服務。 當您在建立具名服務時，指定想要讓狀態分散到多少個資料分割 (提供延展性)。 也請指定跨節點覆寫狀態的次數 (提供可靠性)。 每個具名服務都有一個主要複本和多個次要複本。 您可以在寫入主要複本時修改具名服務的狀態。 然後，Service Fabric 會將此狀態複寫至所有次要複本以保持狀態同步。當主要複本失敗時，Service Fabric 會自動偵測到此狀況，並將現有的次要複本升級為主要複本。 然後 Service Fabric 會建立新的次要複本。  

**複本和執行個體**會參照正在部署與執行之服務的程式碼 (及具狀態服務的狀態)。 請參閱[複本和執行個體](service-fabric-concepts-replica-lifecycle.md)。

**重新設定**是指在服務複本集中進行任何變更的流程。 請參閱[重新設定](service-fabric-concepts-reconfiguration.md)。

**服務套件**：磁碟目錄，包含此服務類型的 `ServiceManifest.xml` 檔案。 這個檔案會參考此服務類型的程式碼、靜態資料和組態封裝。 此應用程式類型的 `ApplicationManifest.xml` 檔會參考此服務封裝目錄中的檔案。 例如，服務套件可能會參考構成資料庫服務的程式碼、靜態資料和設定套件。

**具名服務**：建立具名應用程式之後，可以在叢集內建立應用程式其中一種服務類型的執行個體。 您可以使用它的名稱/版本以指定服務類型。 需為每個服務類型執行個體指派一個 URI (名稱範圍需在其具名應用程式的 URI 之下)。 例如，如果您在具名應用程式 MyNamedApp 內建立 MyDatabase 具名服務，URI 看起來就像這樣： `"fabric:/MyNamedApp/MyDatabase"`。 在具名應用程式中，可以建立數個具名服務。 每個具名服務可以有自己的分割配置和執行個體或複本計數。

**程式碼套件**：磁碟目錄，包含此服務類型的執行檔，通常是 EXE/DLL 檔。 此服務類型的 `ServiceManifest.xml` 檔會參考此程式碼封裝目錄中的檔案。 當您建立具名服務時，會將程式碼套件複製到選取用來執行具名服務的一或多個節點。 然後程式碼會開始執行。 程式碼封裝執行檔分成兩種：

* **來賓執行檔**：在主機作業系統 (Windows 或 Linux) 上以原樣執行的執行檔。 這些可執行檔未連結至或參考任何 Service Fabric 執行階段檔案作業，因此不會使用任何 Service Fabric 程式設計模型。 這些可執行檔無法使用某些 Service Fabric 功能，例如端點探索的命名服務。 來賓可執行檔無法報告每個服務執行個體特定的負載計量。
* **服務主機可執行檔**：藉由連結至 Service Fabric 執行階段檔案的方式使用 Service Fabric 程式設計模型，以支援 Service Fabric 功能的可執行檔。 例如，具名服務執行個體可以在 Service Fabric 命名服務註冊端點，也可以報告負載度量。      

**資料套件**：磁碟目錄，包含此服務類型的靜態唯讀資料檔，通常是相片、音訊和視訊檔。 此服務類型的 `ServiceManifest.xml` 檔會參考此資料封裝目錄中的檔案。 當您建立具名服務時，會將資料套件複製到選取用來執行具名服務的一或多個節點。 程式碼會開始執行，此時即可存取資料檔案。

**設定套件**：磁碟目錄，包含此服務類型的靜態唯讀設定檔，通常是文字檔。 此服務類型的 `ServiceManifest.xml` 檔會參考此組態封裝目錄中的檔案。 當您建立具名服務時，系統會將設定套件中的檔案，複製到一或多個選取用來執行具名服務的節點。 接著會開始執行程式碼，此時即可存取設定檔。

**容器**：根據預設，Service Fabric 會以處理序形式部署和啟動這些服務。 Service Fabric 也可以在容器映像中部署服務。 容器是從應用程式中將基礎作業系統虛擬化的一種虛擬化技術。 應用程式及其執行階段、相依性和系統程式庫會在容器內執行。 此容器有容器專屬之作業系統建構檢視的完整、私人存取權。 Service Fabric 支援 Linux 上的 Docker 容器和 Windows Server 容器。 如需詳細資訊，請參閱 [Service Fabric 和容器](service-fabric-containers-overview.md)。

**分割配置**：當您建立具名服務時，指定分割配置。 含有大量狀態的服務會跨分割切割其資料，所以服務是將狀態分散在叢集的節點上。 藉由跨分割切割資料，您的具名服務狀態可以調整。 在分割內，無狀態的具名服務會有執行個體，而具狀態的具名服務則有複本。 通常，無狀態具名服務只會有 1 個分割，因為它們有沒有內部狀態。 資料分割執行個體提供可用性。 若某個執行個體失敗，其他執行個體會繼續正常運作，接著 Service Fabric 會建立新的執行個體。 具狀態的具名服務會在複本中維持其狀態，且每個分割都有自己的複本集，其中包含保持同步的所有狀態。複本失敗失敗時，Service Fabric 會從現有複本建立新的複本。

如需詳細資訊，請閱讀 [分割 Service Fabric Reliable Services](service-fabric-concepts-partitioning.md) 。

## <a name="system-services"></a>系統服務
有在每個叢集中建立的系統服務，用來提供 Service fabric 的平台功能。

**命名服務**：每個 Service Fabric 叢集有一個命名服務，此服務會將服務名稱解析至叢集中的位置。 您可以管理服務名稱與屬性，類似叢集的網際網路網域名稱系統 (DNS)。 用戶端可以使用命名服務，與叢集中的任何節點安全地進行通訊，以便解析服務名稱及其位置。 應用程式會在叢集內移動。 例如，由於失敗、平衡資源、或調整叢集大小。 您可以開發可解析目前網路位置的服務和用戶端。 用戶端會取得實際的電腦 IP 位址，以及目前執行所在的連接埠。

如需使用搭配命名服務運作的用戶端與服務通訊 API 詳細資訊，請閱讀[與服務通訊](service-fabric-connect-and-communicate-with-services.md)一文。

**映像存放區服務**︰每個 Service Fabric 叢集都有一個映像存放區服務，其中保存已部署且版本化的應用程式套件。 將應用程式封裝複製到映像存放區，然後註冊該應用程式封裝內包含的應用程式類型。 佈建應用程式類型後，您可以從中建立具名應用程式。 在刪除應用程式類型的所有具名應用程式之後，可以從映像存放區服務取消註冊該應用程式類型。

如需映像存放區服務的詳細資訊，請參閱[了解 ImageStoreConnectionString 設定](service-fabric-image-store-connection-string.md)。

如需有關將應用程式部署至映像存放區服務的詳細資訊，請閱讀[部署應用程式](service-fabric-deploy-remove-applications.md)一文。

**容錯移轉管理員服務**：每個 Service Fabric 叢集有容錯移轉管理員服務會負責下列動作：
   - 執行與服務高可用性和一致性相關的功能。
   - 協調應用程式和叢集升級。
   - 與其他系統元件互動。

**Repair Manager 服務**：這是選擇性的系統服務，讓您以安全、自動化及透明的方式針對叢集執行修復動作。 Repair Manager 適用於：
   - 針對 [Silver 和 Gold 持久性](service-fabric-cluster-capacity.md#the-durability-characteristics-of-the-cluster) Azure Service Fabric 叢集執行 Azure 維護修復作業。
   - 針對[修補程式協調流程應用程式](service-fabric-patch-orchestration-application.md)執行修復動作

## <a name="built-in-programming-models"></a>內建的程式設計模型
有一些 .NET Framework 和 Java 程式設計模型可讓您建置 Service Fabric 服務：

**Reliable Services**：用於建置無狀態和具狀態服務的 API。 具狀態服務將其狀態儲存在 Reliable Collections 中，例如字典或佇列。 您也可以插入各種不同的通訊堆疊，例如 Web API、Windows Communication Foundation (WCF)。

**Reliable Actors**：可透過虛擬的 Actor 程式設計模型，建置無狀態和具狀態物件的 API。 當您有多個獨立的計算/狀態單位時，此模型相當實用。 此模型使用回合式執行緒模型，因此最好避免向外呼叫其他動作項目或服務的程式碼，因為直到個別動作項目的所有輸出要求完成，動作項目才能處理其他連入要求。

您也可以在 Service Fabric 上執行現有的應用程式：

：Service Fabric 支援將 Docker 容器部署至 Linux，也支援將 Windows Server 容器部署至 Windows Server 2016，並支援 Hyper-V 隔離模式。 在 Service Fabric [應用程式模型](service-fabric-application-model.md)中，容器代表多個服務複本所在的應用程式主機。 Service Fabric 可以執行任何容器，其案例類似於來賓可執行檔案例，可讓您封裝容器中的現有應用程式。 此外，您也可以執行[容器內的 Service Fabric 服務](service-fabric-services-inside-containers.md)。

**來賓可執行檔**：您可以在 Azure Service Fabric 中將任何類型的程式碼 (例如 Node.js、Java 或 C++) 當作服務來執行。 Service Fabric 將這些類型的服務稱為來賓可執行檔，並且視為無狀態服務。 在 Service Fabric 叢集中執行來賓可執行檔的 優點包括高可用性、健康情況監控、應用程式生命週期管理、高密度及可搜尋性。

如需詳細資訊，請閱讀[為服務選擇程式設計模型](service-fabric-choose-framework.md)一文。

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## <a name="next-steps"></a>後續步驟
若要深入了解 Service Fabric：

* [Service Fabric 概觀](service-fabric-overview.md)
* [為何要用微服務方式建置應用程式？](service-fabric-overview-microservices.md)
* [應用程式案例](service-fabric-application-scenarios.md)


