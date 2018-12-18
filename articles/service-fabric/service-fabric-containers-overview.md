---
title: Service Fabric 和容器的概觀 | Microsoft Docs
description: Service Fabric 及使用容器來部署微服務應用程式的概觀。 本文提供如何使用容器及 Service Fabric 所提供之功能的概觀。
services: service-fabric
documentationcenter: .net
author: msfussell
manager: timlt
editor: ''
ms.assetid: c98b3fcb-c992-4dd9-b67d-2598a9bf8aab
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 8/8/2018
ms.author: msfussell
ms.openlocfilehash: 6715142be7f40955861afa634bf6e2472c9f7294
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40005770"
---
# <a name="service-fabric-and-containers"></a>Service Fabric 和容器

## <a name="introduction"></a>簡介

Azure Service Fabric 是一個分散式系統平台，可讓您輕鬆封裝、部署及管理可調整和可信賴的微服務與容器。

Service Fabric 是將微服務部署至整個機器叢集的 Microsoft [容器協調者](service-fabric-cluster-resource-manager-introduction.md)。 Service Fabric 也從 Microsoft 大規模服務執行的數年期間受惠不少。

微服務的開發方式有許多種，從使用 [Service Fabric 程式設計模型](service-fabric-choose-framework.md)、[ASP.NET Core](service-fabric-reliable-services-communication-aspnetcore.md)，到部署[您選擇的任何程式碼](service-fabric-guest-executables-introduction.md)，應有盡有。 或者，如果您只想[部署和管理容器](service-fabric-containers-overview.md)，Service Fabric 也是一個絕佳選擇。

根據預設，Service Fabric 會以處理序形式部署和啟動這些服務。 處理序能以最快速度和最密集的程度使用叢集的資源。 Service Fabric 也可以在容器映像中部署服務。 您也可以在相同應用程式中，混合容器中以及處理序中的服務。

若要直接進入正題並試用 Service Fabric 上的容器，請嘗試快速入門、教學課程或範例：  

[快速入門：將 Linux 容器應用程式部署到 Service Fabric](service-fabric-quickstart-containers-linux.md)  
[快速入門：將 Windows 容器應用程式部署到 Service Fabric](service-fabric-quickstart-containers.md)  
[將現有的 .NET 應用程式容器化](service-fabric-host-app-in-a-container.md)  
[Service Fabric 容器範例](https://azure.microsoft.com/resources/samples/service-fabric-containers/)  

## <a name="what-are-containers"></a>何謂容器

容器會提供固定環境以供應用程式在其中執行，進而可靠地解決在不同的運算環境中執行應用程式的問題。 容器會將應用程式和其所有相依性 (例如程式庫和組態檔) 包裝成自有的隔離「方塊」，其中包含在容器內執行軟體所需的一切。 每當容器執行時，其中的應用程式一律有它所需執行的一切，例如其相依程式庫的正確版本、任何組態檔案，以及它所需執行的任何其他項目。

容器直接執行於核心之上，而且在檔案系統和其他資源中有其各自隔離的範圍。 容器中的應用程式並不知道其容器外部的任何其他應用程式或程序。 每個應用程式與其執行階段、相依性及系統程式庫在容器內執行時，在其各自於作業系統中的獨立範圍內，都具有容器的完整專屬存取權。 除了讓您輕鬆地提供應用程式在不同運算環境中執行所需的所有相依性，安全性和資源隔離也是使用容器搭配 Service Fabric 的重要優勢 -- 否則會在一個程序中執行多項服務。

相較於虛擬機器，容器的優點如下︰

* **小**︰容器使用單一儲存空間和層級版本與更新，以增加效率。
* **快**︰容器不需要啟動整個作業系統，因此可以更快啟動，通常差個幾秒。
* **可攜性**︰容器化應用程式映像可以移植到雲端或內部部署中，而在虛擬機器內或直接在實體機器上執行。
* **資源管理**：容器可以限制在其主機上可取用的實體資源。

### <a name="container-types-and-supported-environments"></a>容器類型和支援的環境

Service Fabric 支援 Linux 和 Windows 上的容器，也支援 Windows 上的 Hyper-V 隔離模式。

#### <a name="docker-containers-on-linux"></a>Linux 上的 Docker 容器

Docker 提供 API 來建立和管理 Linux 核心容器頂端的容器。 Docker 中樞提供中央存放庫來儲存和擷取容器映像。
如需以 Linux 為基礎的教學課程，請參閱[在 Linux 建立第一個 Service Fabric 容器應用程式](service-fabric-get-started-containers-linux.md)。

#### <a name="windows-server-containers"></a>Windows Server 容器

Windows Server 2016 提供兩種不同的容器，其隔離程度有所不同。 Windows Server 容器類似於 Docker 容器，因為都有命名空間和檔案系統隔離，然而與它們執行所在的主機共用核心。 在 Linux 上，傳統上由控制群組和命名空間提供這種隔離，而 Windows Server 容器具有類似的行為。

含 Hyper-V 支援的 Windows 容器提供更高程度的隔離和安全性，因為容器彼此之間或與主機之間不會共用作業系統核心。 由於有如此高程度的安全性隔離，已啟用 Hyper-V 的容器在潛在惡意多租用戶的情況下是重點目標。
如需以 Windows 為基礎的教學課程，請參閱[在 Windows 建立第一個 Service Fabric 容器應用程式](service-fabric-get-started-containers.md)。

下圖顯示可用的各種不同虛擬化及隔離等級。
![Service Fabric 平台][Image1]

## <a name="scenarios-for-using-containers"></a>容器使用案例

以下是典型範例，容器是很好的選擇︰

* **IIS 提起然後平移**︰您可以將現有的 [ASP.NET MVC](https://www.asp.net/mvc) 應用程式放在一個容器中，而不是將它遷移到 ASP.NET Core。 這些 ASP.NET MVC 應用程式相依於網際網路資訊服務 (IIS)。 您可以從預先建立的 IIS 映像將這些應用程式封裝成容器映像，然後與 Service Fabric 一起部署。 請參閱 [Windows Server 上的容器映像](https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-server) (英文)，以取得 Windows 容器的相關資訊。

* **混合容器和 Service Fabric 微服務**︰針對您的應用程式的一部分使用現有的容器映像。 例如，對於應用程式的 Web 前端系統，您可以使用 [NGINX 容器](https://hub.docker.com/_/nginx/)，而對於更密集的後端運算，則可以使用具狀態服務。

* **減少「壟斷」服務的影響**︰您可以使用容器的資源控管能力來限制服務在主機上使用的資源。 如果服務可能會耗用大量資源，因而影響其他服務的效能 (例如，像作業一樣長時間執行的查詢)，請考慮將這些服務放到可控管資源的容器中。

## <a name="service-fabric-support-for-containers"></a>Service Fabric 的容器支援

Service Fabric 支援將 Docker 容器部署至 Linux，也支援將 Windows Server 容器部署至 Windows Server 2016，並支援 Hyper-V 隔離模式。 

Service Fabric 會提供[應用程式模型](service-fabric-application-model.md)，其中的容器代表多個服務複本所在的應用程式主機。 Service Fabric 也支援[來賓可執行案例](service-fabric-guest-executables-introduction.md)，這此案例中，您不會使用內建 Service Fabric 程式設計模型，但是會封裝容器內使用任何語言或架構撰寫的現有應用程式。 此案例是容器常見的使用案例。

您也可以執行[容器內的 Service Fabric 服務](service-fabric-services-inside-containers.md)。 執行容器內 Service Fabric 服務的支援有限。

Service Fabric 提供數個容器功能，協助您建置由容器化微服務組成的應用程式，例如：

* 容器映像部署和啟用。
* 資源管理，包括依預設設定 Azure 叢集上的資源值。
* 儲存機制驗證。
* 容器連接埠至主機連接埠的對應。
* 容器至容器的探索及通訊。
* 能夠設定環境變數。
* 能夠在容器上設定安全性認證。
* 為容器選擇不同的網路模式。

如需在 Azure 上支援容器的完整概觀，例如如何利用 Azure Kubernetes 服務建立 Kubernetes 叢集、如何在 Azure Container Registry 中建立私人 Docker 登錄等等，請參閱 [Azure 容器](https://docs.microsoft.com/azure/containers/)。

## <a name="next-steps"></a>後續步驟

在本文中，您已了解 Service Fabric 提供給執行中容器的支援。 接下來，我們將示範每一項功能來展示其用法。

[在 Linux 建立第一個 Service Fabric 容器應用程式](service-fabric-get-started-containers-linux.md)  
[在 Windows 建立第一個 Service Fabric 容器應用程式](service-fabric-get-started-containers.md)  
[深入了解 Windows 容器](https://docs.microsoft.com/virtualization/windowscontainers/about/)

[Image1]: media/service-fabric-containers/Service-Fabric-Types-of-Isolation.png