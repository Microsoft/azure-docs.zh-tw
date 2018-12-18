---
title: Azure App Service、虛擬機器、Service Fabric 及雲端服務的比較 | Microsoft Docs
description: 了解如何在 Azure App Service、虛擬機器、Service Fabric 及雲端服務之間做選擇以供裝載 Web 應用程式。
services: app-service\web, virtual-machines, cloud-services
documentationcenter: ''
author: cephalin
manager: jeconnoc
ms.assetid: 7d346a23-532a-42a9-98a8-23b7286d32a8
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 07/07/2016
ms.author: cephalin
ms.custom: mvc
ms.openlocfilehash: 3b1fb14fbb21876d0b3f7d98327353d54bb1cfb2
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42022671"
---
# <a name="azure-app-service-virtual-machines-service-fabric-and-cloud-services-comparison"></a>Azure App Service、虛擬機器、Service Fabric 及雲端服務的比較

Azure 提供數種託管網站的方式：[Azure App Service][Azure App Service]、[虛擬機器][Virtual Machines]、[Service Fabric][Service Fabric] 及[雲端服務][Cloud Services]。 本文協助您了解這些選項，為您的 Web 應用程式做出正確的選擇。

Azure App Service 是大多數 Web 應用程式的最佳選擇。 部署和管理都已整合到平台，網站可以迅速調整規模以因應過高的流量負載，而內建的負載平衡和流量管理員提供高可用性。 您可以使用[線上移轉工具][migrate-tool]輕鬆地將現有網站移至 Azure App Service、使用 Web 應用程式庫中的開放原始碼應用程式，或使用您選擇的架構和工具來建立新網站。 [WebJobs][WebJobs] 功能可讓您輕鬆地將背景作業處理新增至 App Service Web 應用程式。

如果您在建立新的應用程式或重新撰寫現有的應用程式以使用微服務架構，Service Fabric 是不錯的選擇。 在共用機器集區上執行的應用程式可以從小規模著手，然後擴充為包含成千上萬部機器的大規模服務。 具狀態服務可輕鬆地以一致且可靠的方式儲存應用程式狀態，而 Service Fabric 會自動為您管理服務資料分割、調整及可用性。  Service Fabric 也支援具有 Open Web Interface for .NET (OWIN) 和 ASP.NET Core 的 WebAPI。  相較於 App Service，Service Fabric 也能更充分掌控或直接存取基礎結構。 您可以從遠端進入您的伺服器，或設定伺服器啟動工作。 雲端服務的控制和易用程度類似於 Service Fabric，但是它現在是舊版服務，建議將 Service Fabric 用於新的開發。

如果現有的應用程式需要進行大幅修改才能在 App Service 或 Service Fabric 中執行，您可以選擇 [虛擬機器] 以簡化移轉至雲端的工作。 不過，相較於 Azure App Service 和 Service Fabric，正確設定、保護和維護 VM 需要投入更多時間和 IT 專業知識。 如果您考慮採用 Azure 虛擬機器，請確定您已將修補、更新和管理 VM 環境所需的持續性維護工作都納入考量。 Azure 虛擬機器是基礎結構即服務 (IaaS)，而 App Service 和 Service Fabric 是平台即服務 (Paas)。 

## <a name="features"></a>功能比較
下表比較 App Service、雲端服務、虛擬機器及 Service Fabric 的功能，以協助您做出最佳選擇。 如需每個選項目前的 SLA 資訊，請參閱＜ [Azure 服務等級協定](https://azure.microsoft.com/support/legal/sla/)＞。

| 功能 | App Service (Web 應用程式) | 雲端服務 (Web 角色) | 虛擬機器 | Service Fabric | 注意 |
| --- | --- | --- | --- | --- | --- |
| 近乎即時的部署 |X | | |X |將應用程式或應用程式更新部署到雲端服務，或建立 VM，至少需要幾分鐘的時間；將應用程式部署到 Web 應用程式只需要幾秒鐘。 |
| 向上調整至更大的機器而不重新部署 |X | | |X | |
| Web 伺服器執行個體會共用內容和組態，這表示您在調整規模時不需要重新部署或重新設定。 |X | | |X | |
| 多個部署環境 (生產環境和預備環境) |X |X | |X |Service Fabric 可讓您擁有各種應用程式適用的環境，或並列部署您應用程式的不同版本。 |
| 自動管理 OS 更新 |X |X | | |部分透過修補程式協調流程應用程式 (POA)，未來則是完全。 |
| 順暢切換平台 (輕鬆地切換到 32 位元或 64 位元) |X |X | | | |
| 利用 GIT、FTP 來部署程式碼 |X | |X | | |
| 利用 Web Deploy 來部署程式碼 |X | |X | |雲端服務支援使用 Web Deploy 將更新部署到個別角色的執行個體。 不過，您無法將它用於角色的初始部署，如果使用 Web Deploy 來部署更新，則必須分別地部署到角色的每一個執行個體。 需要有多個執行個體，才能符合生產環境的雲端服務 SLA。 |
| 支援 WebMatrix |X | |X | | |
| 存取服務匯流排、儲存體、SQL Database 等服務。 |X |X |X |X | |
| 裝載多層式架構的 Web 或 Web 服務層 |X |X |X |X | |
| 裝載多層式架構的中間層 |X |X |X |X |App Service Web 應用程式可以輕易裝載 REST API 中間層，而 [WebJobs](http://go.microsoft.com/fwlink/?linkid=390226) 功能可以裝載背景處理工作。 您可以在專用網站中執行 WebJobs，以實現此層的獨立擴充性。 |
| 整合 MySQL 即服務的支援 |X |X | | | |
| 支援 ASP.NET、傳統 ASP、Node.js、PHP、Python |X |X |X |X |Service Fabric 支援使用 [ASP.NET 5](../service-fabric/service-fabric-reliable-services-communication-aspnetcore.md) 建立 Web 前端，也可讓您以[來賓可執行檔](../service-fabric/service-fabric-guest-executables-introduction.md)的形式部署任何類型的應用程式 (Node.js、Java 等)。 |
| 向外延展至多個執行個體而不重新部署 |X |X |X |X |「虛擬機器」可向外延展至多個執行個體，但這些機器上執行的服務必須設計成應付這個向外延展情況。您必須設定負載平衡器以在機器間路由要求，並確定您在[可用性設定組](../virtual-machines/windows/manage-availability.md)中有多個 VM 執行個體。 |
| 支援 SSL |X |X |X |X |在 App Service Web 應用程式中，只有基本和標準模式才支援自訂網域名稱的 SSL。 如需 Web 應用程式使用 SSL 的相關資訊，請參閱[設定 Azure 網站的 SSL 憑證](app-service-web-tutorial-custom-ssl.md)。 |
| 整合 Visual Studio |X |X |X |X | |
| 遠端偵錯 |X |X |X | | |
| 利用 TFS 來部署程式碼 |X |X |X |X | |
| 利用 [Azure 虛擬網路](/azure/virtual-network/) |X |X |X |X |另請參閱＜ [Azure 網站虛擬網路整合](https://azure.microsoft.com/blog/2014/09/15/azure-websites-virtual-network-integration/) |
| 支援 [Azure 流量管理員](/azure/traffic-manager/) |X |X |X |X | |
| 整合式端點監視 |X |X |X | | |
| 透過桌面遠端來存取伺服器 | |X |X |X | |
| 安裝任何自訂 MSI | |X |X |X |Service Fabric 可讓您以 [來賓可執行檔](../service-fabric/service-fabric-guest-executables-introduction.md) 的形式託管任何可執行檔，也可讓您在 VM 上安裝任何應用程式。 |
| 能夠定義/執行啟動工作 | |X |X |X | |
| 可接聽 ETW 事件 | |X |X |X | |

## <a name="scenarios"></a>案例和建議
關於哪個 Azure Web 裝載選項最適合，以下是一些常見的應用程式案例與建議。

* [我需要一個具有背景處理和資料庫後端的 Web 前端，以執行與內部部署資源整合的商業應用程式。](#onprem)
* 
  [我需要可靠的方法來裝載可靈活調整且可供全球存取的公司網站。](#corp)
* [我有一個在 Windows Server 2003 上執行的 IIS6 應用程式。](#iis6)
* [我是小型企業業主，需要有便宜的方式來裝載網站，但又想顧及網站未來成長的可能性。](#smallbusiness)
* [我是網頁或平面設計師，想要幫客戶設計及建立網站。](#designer)
* [我想將含 Web 前端的多層式應用程式移轉至雲端。](#multitier)
* [我的應用程式需要在高度自訂的 Windows 或 Linux 環境中才能運作，我想要將它移轉至雲端。](#custom)
* [我的網站使用開放原始碼軟體，我想要將該軟體裝載在 Azure 上。](#oss)
* [我有個主要商務應用程式需要連線至公司網路。](#lob)
* [我想要裝載 REST API 或 Web 服務供行動用戶端使用。](#mobile)

### <a id="onprem"></a>我需要一個具有背景處理和資料庫後端的 Web 前端，以執行與內部部署資源整合的商業應用程式。
Azure App Service 是複雜商業應用程式的絕佳解決方案。 您開發的應用程式將能夠在負載平衡平台上自動調整、採用 Active Directory 來保護，以及連接到內部部署資源。 它可讓您透過世界級的入口網站和 API 來輕鬆管理應用程式，並利用應用程式洞察工具來深入了解客戶如何使用應用程式。 [Webjobs][Webjobs] 功能可讓您將背景程序和工作當作 Web 層的一部分執行，而混合式連線和 VNET 功能可讓您輕鬆地連回內部部署資源。 Azure App Service 為 Web 應用程式提供三個 9 的 SLA，可讓您：

* 在自我修復、自動修補的雲端平台上可靠地執行應用程式。
* 在全球的資料中心網路上自動調整。
* 針對災害復原進行備份與還原。
* 符合 ISO、SOC2 和 PCI。
* 與 Active Directory 整合

### 
  <a id="corp">
  </a> 我需要可靠的方法來裝載可靈活調整且可供全球存取的公司網站。
Azure App Service 是裝載公司網站的絕佳解決方案。 它可讓 Web 應用程式快速調整，輕鬆符合全球資料中心網路的需求。 它提供本機存取、容錯和智慧型流量管理。 一切盡在一個提供世界級管理工具的平台上，可讓您快速又輕鬆地深入了解網站健康情況和網路流量。 Azure App Service 為 Web 應用程式提供三個 9 的 SLA，可讓您：

* 在自我修復、自動修補的雲端平台上可靠地執行網站。
* 在全球的資料中心網路上自動調整。
* 針對災害復原進行備份與還原。
* 使用整合式工具來管理記錄和流量。
* 符合 ISO、SOC2 和 PCI。
* 與 Active Directory 整合

### <a id="iis6"></a> 我有一個在 Windows Server 2003 上執行的 IIS6 應用程式。
Azure App Service 可讓您輕鬆地省去移轉舊版 IIS6 應用程式時相關的基礎結構成本。 Microsoft 建立[簡單易用的移轉工具和詳細的移轉指引][migrate-tool]，可讓您檢查相容性和識別任何需要進行的變更。 與 Visual Studio、TFS 和一般 CMS 工具整合，讓您輕鬆地將 IIS6 應用程式直接部署到雲端。 部署之後，Azure 入口網站提供健全的管理工具，可讓您依需要相應減少來管理成本，或相應增加來符合需求。 移轉工具可讓您：

* 快速又輕鬆地將舊式的 Windows Server 2003 Web 應用程式移轉至雲端。
* 選擇將連結的 SQL 資料庫留在內部部署，以建立混合式應用程式。
* 自動隨著舊式應用程式一起移動 SQL 資料庫。

### <a id="smallbusiness"></a>我是小型企業業主，需要有便宜的方式來裝載網站，但又想顧及網站未來成長的可能性。
Azure App Service 是此案例的絕佳解決方案，因為您可先免費使用它，等到需要更多功能時再新增功能即可。 每一個免費 Web 應用程式都隨附 Azure 提供的網域 (your_company.azurewebsites.net)，且平台包含整合式部署和管理工具及應用程式資源庫，輕鬆地就能開始使用。 另外還有其他許多服務與調整選項，可讓網站隨使用者需求的增加來發展。 使用 Azure App Service，您可以：

* 從免費版本開始，視需要進行向上調整。
* 使用應用程式庫迅速設定熱門的 Web 應用程式，例如 WordPress。
* 視需要新增其他 Azure 服務與功能至您的應用程式。
* 使用 HTTPS 來保護 Web 應用程式。

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

### <a id="designer"></a> 我是網頁或平面設計師，想要幫客戶設計及建立網站
對於 Web 開發人員和設計人員，Azure App Service 可輕鬆整合各種架構和工具、提供 Git 和 FTP 的部署支援，並與 Visual Studio 和 SQL Database 等工具和服務密切整合。 使用 App Service，您可以：

* 使用命令列工具執行[自動化工作][scripting]。
* 使用熱門語言，例如 [.Net][dotnet]、[PHP][PHP]、[Node.js][nodejs] 及 [Python][Python]。
* 有三種不同的調整層級可選，以向上調整至非常高的容量。
* 與其他 Azure 服務 (如 [SQL Database][sqldatabase]、[服務匯流排][servicebus]及[儲存體][Storage])，或 [Azure 市集][azurestore]上的合作夥伴供應項目 (如 MySQL 和 MongoDB) 整合。
* 與 Visual Studio、Git、WebMatrix、WebDeploy、TFS 和 FTP 等工具整合。

### <a id="multitier"></a>我想將含 Web 前端的多層式應用程式移轉至雲端
如果您執行多層式應用程式，例如連接到資料庫的 Web 伺服器，則 Azure App Service 是適合的選項，能夠與 Azure SQL Database 密切整合。 而您可以使用 WebJobs 功能來執行背景程序。

如果您需要加強控制伺服器環境，例如想要從遠端登入伺服器或設定伺服器啟動工作，請為您的一個或多個層選擇 Service Fabric。

如果您要使用自己的機器映像，或執行無法在 Service Fabric 上設定的軟體或服務，請為您的一個或多個層選擇虛擬機器。

### <a id="custom"></a>我的應用程式需要在高度自訂的 Windows 或 Linux 環境中才能運作，我想要將它移轉至雲端。
如果您的應用程式需要用到安裝或設定方式複雜的軟體與作業系統，則「虛擬機器」可能是最佳解決方案。 虛擬機器可讓您：

* 使用虛擬機器庫來得到初步的作業系統，例如 Windows 或 Linux，然後根據自己的應用程式需求加以自訂。
* 建立並上傳現有內部部署伺服器的自訂映像，使其執行於 Azure 中的虛擬機器上。

### <a id="oss"></a>我的網站使用開放原始碼軟體，我想要將該軟體裝載在 Azure 上。
如果 App Service 上支援您的開放原始碼架構，則會自動設定您的應用程式所需的語言和架構。 App Service 可讓您：

* 使用許多熱門的開放原始碼語言，例如 [.NET][dotnet]、[PHP][PHP]、[Node.js][nodejs] 和 [Python][Python]。
* 設定 WordPress、Drupal、Umbraco、DNN 及其他許多協力廠商 Web 應用程式。
* 移轉現有的應用程式，或從應用程式庫建立新的應用程式。

如果 App Service 上不支援您的開放原始碼架構，您可以在其中一個其他 Azure Web 裝載選項上執行它。 「虛擬機器」則需要您在機器映像 (可以是 Windows 或 Linux 型) 上安裝並設定軟體。

### <a id="lob"></a>我有個主要商務應用程式需要連線至公司網路。
若要建立企業營運系統應用程式，您的網站可能需要直接存取公司網路上的服務或資料。 在 App Service、Service Fabric和虛擬機器上利用 [Azure 虛擬網路服務](/azure/virtual-network/)即可達成。 在 App Service 上，您可以使用 [VNET 整合功能](https://azure.microsoft.com/blog/2014/09/15/azure-websites-virtual-network-integration/)，讓您的 Azure 應用程式彷彿就像在公司網路上執行一樣。

### <a id="mobile"></a>我想要裝載 REST API 或 Web 服務供行動用戶端使用
HTTP 型 Web 服務可讓您支援各種用戶端，包括行動用戶端。 ASP.NET Web API 等架構會與 Visual Studio 整合，讓您更容易建立及取用 REST 服務。  這些服務是透過 Web 端點公開，因此在 Azure 上可以使用任何 Web 裝載技術來支援此案例。 不過，App Service 是裝載 REST API 的絕佳選擇。 使用 App Service，您可以：

* 迅速建立一個[行動應用程式](../app-service-mobile/app-service-mobile-value-prop.md)或 API 應用程式，以將 HTTP Web 服務裝載於 Azure 全球各地的其中一個資料中心內。
* 移轉現有的服務或建立新服務。
* 以單一執行個體達到可用性 SLA，或向外延展至多部專用機器。
* 使用已發佈的網站來提供 REST API 給任何 HTTP 用戶端，包括行動用戶端。

> [!NOTE]
> 如果您要在註冊帳戶前開始使用 Azure App Service，請移至 <a href="https://trywebsites.azurewebsites.net/">https://trywebsites.azurewebsites.net</a>，您可以在 Azure App Service 中立即建立短期的免費簡易版應用程式。 不需要信用卡，沒有承諾。
> 
> 

## <a id="nextsteps"></a> 後續步驟
如需有關這三個 Web 主控選項的詳細資訊，請參閱 [Azure 簡介](../fundamentals-introduction-to-azure.md)。

若要開始對應用程式使用所選擇的選項，請參閱下列資源：

* [Azure App Service](/azure/app-service/)
* [Azure 雲端服務](/azure/cloud-services/)
* [Azure 虛擬機器](/azure/virtual-machines/)
* [Service Fabric](/azure/service-fabric/)

<!-- URL List -->

[Azure App Service]: /azure/app-service/
[Cloud Services]: /azure/cloud-services/
[Virtual Machines]: /azure/virtual-machines/
[Service Fabric]: /azure/service-fabric/
[WebJobs]: http://go.microsoft.com/fwlink/?linkid=390226&clcid=0x409
[Configuring an SSL certificate for an Azure Website]: app-service-web-tutorial-custom-ssl.md
[azurestore]: https://azuremarketplace.microsoft.com/en-us/marketplace/apps
[scripting]: https://azure.microsoft.com/documentation/scripts/?services=web-sites
[dotnet]: https://azure.microsoft.com/develop/net/
[nodejs]: https://azure.microsoft.com/develop/nodejs/
[PHP]: https://azure.microsoft.com/develop/php/
[Python]: https://azure.microsoft.com/develop/python/
[servicebus]: /azure/service-bus/
[sqldatabase]: /azure/sql-database/
[Storage]: /azure/storage/

<!-- IMG List -->

[ChoicesDiagram]: ./media/choose-web-site-cloud-service-vm/Websites_CloudServices_VMs_3.png
[migrate-tool]: https://www.movemetothecloud.net/
