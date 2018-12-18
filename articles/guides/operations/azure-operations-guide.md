---
title: Azure IT 操作員快速入門指南 | Microsoft Docs
description: Azure IT 操作員快速入門指南
services: ''
documentationcenter: ''
author: themichaelbender-ms
manager: timlt
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.devlang: ''
ms.topic: ''
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 08/24/2018
ms.author: mibender
ms.openlocfilehash: c10e071100410f6d5d995d88657bdd85c08a9715
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47423488"
---
# <a name="get-started-for-azure-it-operators"></a>Azure IT 操作員快速入門

本指南介紹與 Microsoft Azure 基礎結構部署和管理相關的核心概念。 如果您不熟悉雲端運算或 Azure 本身，本指南可協助您快速了解概念、部署及管理詳細資料。 本指南的許多小節討論部署虛擬機器等作業，並接著提供深入技術詳細資料的連結。

## <a name="cloud-computing-overview"></a>雲端運算概觀

雲端運算對傳統內部部署資料中心提供新式替代選項。 公用雲端供應商會提供和管理所有運算基礎結構和基礎管理軟體。 這些廠商會提供各種雲端服務。 在此情況下，雲端服務可能是虛擬機器、Web 伺服器或雲端裝載的資料庫引擎。 身為雲端提供者的客戶，您可視需要租用這些雲端服務。 這麼一來，您可將硬體維護的資本費用轉換成作業費用。 雲端服務也提供下列優勢：

- 快速部署或大量計算環境

- 快速解除配置不再需要的系統

- 輕鬆部署傳統上複雜的系統，像是負載平衡器

- 能夠提供彈性計算能力，或在需要時進行調整

- 更符合成本效益的運算環境

- 透過網頁型入口網站或以程式設計的自動化功能從任何地方進行存取

- 滿足大部分計算和應用程式需求的雲端服務

在內部部署基礎結構內，您必須完全掌控已部署的硬體和軟體。 過去這會導致硬體採購決策著重於相應增加。 例如，購買更多核心的伺服器，以符合尖峰效能需求。 不幸的是，這個基礎結構在需求期間以外可能並未充份使用。 採用 Azure，您可以只部署您需要的基礎結構，並隨時進行調整。 這會導致將重點放在透過部署額外的計算節點來相應放大，以滿足效能需求。 比起透過昂貴硬體相應增加，相應放大雲端服務更符合成本效益。

Microsoft 已在世界各地部署許多 Azure 資料中心，且更有計劃。 此外，Microsoft 逐漸增加中國和德國等區域中的統領雲端。 只有最大型全域企業可以用這種方式部署資料中心，所以使用 Azure 可讓任何規模的企業輕鬆將其服務部署到客戶的附近。

對於小型企業，Azure 提供低成本的進入點，以及隨著計算需求增加而迅速調整的能力。 這可防止基礎結構方面的大型前期資金投資，並提供視需要設計和重新設計系統的彈性。 雲端運算最適合用於具速錯機制 (Fail-fast) 的初創成長模型。

如需可用 Azure 區域的詳細資訊，請參閱 [Azure 區域](https://azure.microsoft.com/regions/)。

### <a name="cloud-computing-model"></a>雲端運算模型

Azure 會根據對客戶所提供的服務類別來使用雲端運算模型。 三個服務類別包括基礎結構即服務 (IaaS)、平台即服務 (PaaS) 以及軟體即服務 (Saas)。 針對上述各個類別中的運算堆疊元件，廠商需分擔部分或所有責任。 讓我們看一下雲端運算的各個類別。
![雲端運算堆疊比較](./media/cloud-computing-comparison.png)

#### <a name="iaas-infrastructure-as-a-service"></a>IaaS：基礎結構即服務

IaaS 雲端廠商會執行及管理所有實體計算資源和必要軟體，以啟用電腦虛擬化。 此服務的客戶會將虛擬機器部署於這些裝載的資料中心。 雖然虛擬機器位於異地資料中心，但 IaaS 取用者可控制作業系統的設定和管理，基礎結構則交由雲端廠商處理。

Azure 包含數個 IaaS 解決方案，其中包括虛擬機器、虛擬機器擴展集，以及相關的網路基礎結構。 一開始將服務移轉至 Azure 時，虛擬機器是個熱門選擇，因為它允許「隨即轉移」移轉模型。 您可以設定一部 VM (例如目前在您的資料中心執行服務的基礎結構)，然後將軟體移轉至新的 VM。 您可能需要進行組態更新，例如其他服務或儲存體的 URL，但您可以用這種方式移轉許多應用程式。

虛擬機器擴展集是以 Azure 虛擬機器為建置基礎，可提供簡單的方式來部署相同 VM 的叢集。 虛擬機器擴展集也支援自動調整，以便視需要自動部署新的 VM。 這讓虛擬機器擴展集成為裝載較高層級微服務計算叢集 (例如 Azure Service Fabric 和 Azure 容器服務) 的理想平台。

#### <a name="paas-platform-as-a-service"></a>PaaS：平台即服務

採用 PaaS，您可將應用程式部署到雲端服務廠商所提供的環境。 廠商會進行所有的基礎結構管理工作，以便您專注於應用程式開發和資料管理。

Azure 提供數個 PaaS 計算供應項目，包括 Azure App Service 與 Azure 雲端服務的 Web Apps 功能 (Web 和背景工作角色)。 在任一情況下，開發人員可使用多種方法來部署其應用程式，而不需要知道任何有關支援它的具體細節。 開發人員不必建立虛擬機器 (VM)，可使用遠端桌面通訊協定 (RDP) 來登入或安裝應用程式。 他們只需按下按鈕 (或接近它)，Microsoft 所提供的工具就會佈建 VM，然後在其上部署並安裝應用程式。

#### <a name="saas-software-as-a-service"></a>SaaS：軟體即服務

SaaS 是集中裝載和管理的軟體。 它通常以多租用戶架構為基礎 — 單一應用程式版本用於所有客戶。 它可以相應放大為多個執行個體，以確保所有位置的最佳效能。 SaaS 軟體通常透過每月或每年訂用帳戶授權。 SaaS 軟體通常透過每月或每年訂用帳戶授權。 SaaS 軟體廠商會負責軟體堆疊的所有元件，因此，您只需要管理所提供的服務。

Microsoft Office 365 是良好的 SaaS 供應項目範例。 訂閱者需每月或每年支付訂閱費用，他們可取得 Microsoft Exchange、Microsoft OneDrive 和 Microsoft Office 套件即服務的其餘部分。 訂閱者一律可取得最新版本，而系統會為您管理 Exchange 伺服器。 相較於每年安裝及升級 Office，這種作法比較經濟實惠。

## <a name="azure-services"></a>Azure 服務

Azure 在其雲端運算平台中提供許多服務。 這些服務包括下列各項。

### <a name="compute-services"></a>計算服務

用於裝載和執行應用程式工作負載的服務：

- Microsoft Azure 虛擬機器 — Linux 和 Windows

- App Services (Web Apps、Mobile Apps、Logic Apps、API Apps 和 Function Apps)

- Azure Batch (適用於大規模的平行和批次計算作業)

- Azure Service Fabric

- Azure Container Service

### <a name="data-services"></a>資料服務

用於儲存和管理資料的服務：

- Azure 儲存體 (由 Azure Blob、佇列、表格和檔案服務所組成)

- 連接字串

- Azure Cosmos DB

- Microsoft Azure StorSimple

- Azure Redis 快取

### <a name="application-services"></a>應用程式服務

用於建置和操作應用程式的服務：

- Azure Active Directory (Azure AD)

- 用於連線分散式系統的 Azure 服務匯流排

- 用於處理巨量資料的 Azure HDInsight

- Azure 排程器

- Azure 媒體服務

### <a name="network-services"></a>網路服務

適用於 Azure 內以及 Azure 與內部部署資料中心之間的網路服務：

- Azure 虛擬網路

- Azure ExpressRoute

- Azure 提供的 DNS

- Azure 流量管理員

- Azure 內容傳遞網路

如需 Azure 服務的詳細文件，請參閱 [Azure 服務文件](https://docs.microsoft.com/azure)。

## <a name="azure-key-concepts"></a>Azure 重要概念

### <a name="datacenters-and-regions"></a>資料中心和區域

Azure 是在全球各地許多地區正式推出的全域雲端平台。 當您在 Azure 中佈建服務、應用程式或 VM 時，系統會要求您選取區域。 選取的區域代表您應用程式執行所在的特定資料中心。 如需詳細資訊，請參閱 [Azure 區域](https://azure.microsoft.com/regions/)。

使用 Azure 的其中一個優點是您可以將應用程式部署至全球各地的各種資料中心。 您選擇的區域可能會影響應用程式的效能。 最好選擇較接近大多數客戶的區域，以減少網路要求延遲。 您也可以選取區域，以符合在某些國家/地區散發應用程式的法律需求。

### <a name="azure-portal"></a>Azure 入口網站

Azure 入口網站是一種 Web 應用程式，可用來建立、管理和移除 Azure 資源和服務。 Azure 入口網站位於 [portal.azure.com](https://portal.azure.com)。 它包含用於管理 Azure 資源的可自訂儀表板和工具。 它也提供帳單和訂用帳戶資訊。 如需詳細資訊，請參閱 [Microsoft Azure 入口網站概觀](https://azure.microsoft.com/documentation/articles/azure-portal-overview/)和[透過入口網站管理 Azure 資源](https://docs.microsoft.com/azure/azure-portal/resource-group-portal)。

### <a name="resources"></a>資源

Azure 資源是已部署至 Azure 訂用帳戶的個別計算、網路、資料或應用程式裝載服務。 虛擬機器、儲存體帳戶或 SQL 資料庫是一些常見的資源。 Azure 服務通常包含數個相關的 Azure 資源。 例如，Azure 虛擬機器可能包含 VM、儲存體帳戶、網路介面卡和公用 IP 位址。 可以用個別或群組方式建立、管理和刪除這些資源。 本指南稍後會更詳細說明 Azure 資源。

### <a name="resource-groups"></a>資源群組

Azure 資源群組是存放 Azure 方案相關資源的容器。 資源群組可以包含解決方案的所有資源，或只包含您要以群組方式管理的資源。 本指南稍後會更詳細說明 Azure 資源群組。

### <a name="resource-manager-templates"></a>Resource Manager 範本

Azure Resource Manager 範本是一個 JavaScript 物件標記法 (JSON) 檔案，可定義一或多個要部署至資源群組的資源。 它也會定義所部署資源之間的相依性。 本指南稍後會詳細說明 Resource Manager 範本。

### <a name="automation"></a>自動化

除了使用 Azure 入口網站建立、管理和刪除資源，您可以使用 PowerShell 或 Azure 命令列介面 (CLI) 自動執行這些活動。

#### <a name="azure-powershell"></a>Azure PowerShell

Azure PowerShell 是一套模組，提供用於管理 Azure 的 Cmdlet。 您可以使用 Cmdlet 來建立、管理和移除 Azure 服務。 Cmdlet 可協助您達到一致、可重複且自動處理的部署。 如需詳細資訊，請參閱 [如何安裝及設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)。

#### <a name="azure-command-line-interface"></a>Azure 命令列介面

Azure 命令列介面是一個工具，可用來從命令列建立、管理和移除 Azure 資源。 Azure CLI 適用於 Linux、Mac OS X 和 Windows。 如需詳細資訊和技術詳細資料，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。

#### <a name="rest-apis"></a>REST API

Azure 是以支援 Azure 入口網站 UI 的一組 REST API 所建置。 也支援其中的大部分 REST API，讓您以程式設計方式從任何啟用網際網路的裝置佈建和管理 Azure 資源和應用程式。 如需詳細資訊，請參閱 [Azure REST SDK 參考](https://docs.microsoft.com/rest/api/index)。

### <a name="azure-cloud-shell"></a>Azure Cloud Shell

系統管理員可以透過稱為 Azure Cloud Shell 的可供瀏覽器存取體驗，來存取 Azure PowerShell 和 Azure CLI。 這個互動式介面會提供彈性的工具，讓 Linux 和 Windows 的系統管理員使用其選擇的命令列介面 (Bash 或 PowerShell)。 若要存取 Azure Cloud Shell，您可以透過入口網站存取、以獨立 Web 介面的形式在 [shell.azure.com](https://shell.azure.com) 存取，或從數個其他存取點來存取。 如需詳細資訊，請參閱 [Azure Cloud Shell 概觀](https://docs.microsoft.com/azure/cloud-shell/overview)。

## <a name="azure-subscriptions"></a>Azure 訂用帳戶

訂用帳戶是連結至 Azure 帳戶之 Azure 服務的邏輯分組。 單一 Azure 帳戶可以包含多個訂用帳戶。 Azure 服務是根據訂用帳戶計費。 Azure 訂用帳戶有可完全掌控訂用帳戶的「帳戶管理員」，以及可控制訂用帳戶中所有服務的「服務管理員」。 如需有關傳統訂用帳戶管理員的資訊，請參閱[新增或變更 Azure 訂用帳戶管理員](../../billing/billing-add-change-azure-subscription-administrator.md)。 除了系統管理員之外，也可以使用[角色型存取控制 (RBAC)](../../role-based-access-control/overview.md) 對個別帳戶授與 Azure 資源的細部控制權。

### <a name="select-and-enable-an-azure-subscription"></a>選取並啟用 Azure 訂用帳戶

您必須先有訂用帳戶，才能使用 Azure 服務。 可用的訂用帳戶類型有好幾種。

**免費帳戶**：[Azure 網站](https://azure.microsoft.com/)上有註冊免費帳戶的連結。 這提供給您 30 天的課程點數，可以試用 Azure 中的任何資源組合。 如果超過點數金額，您的帳戶就會暫止。 在試用結束時，您的服務會解除委任，將無法再運作。 您可以隨時升級至隨用隨付的訂用帳戶。

**MSDN 訂用帳戶**：如果您有 MSDN 訂用帳戶，您每個月可取得特定數量的 Azure 點數。 例如，如果您有 Microsoft Visual Studio Enterprise 與 MSDN 訂用帳戶，您每個月可取得\$150 的 Azure 點數。

如果超過點數金額，您的服務會停用到下個月開始。 您可以關閉消費限制，並新增要扣繳額外費用的信用卡。 其中有些費用會針對 MSDN 帳戶打折。 例如，您需針對執行 Windows Server 的 VM 支付 Linux 價格，而 Microsoft 伺服器 (例如 Microsoft SQL Server) 不另外收費。 這使得 MSDN 帳戶適合用於開發和測試案例。

**BizSpark 帳戶**：Microsoft BizSpark 程式為新創公司帶來許多好處。 其中一個好處是可存取開發和測試環境的所有 Microsoft 軟體 (最多五個 MSDN 帳戶)。 這五個 MSDN 帳戶各可取得 150 美元的 Azure 點數，而有數項 Azure 服務 (例如虛擬機器) 是以減價費率支付。

**隨用隨付**：透過此訂用帳戶，您可將信用卡或金融卡附加至帳戶，以需支付您所用的服務。 如果您是組織，也可獲准開立發票。

**企業合約**：依照企業合約，您承諾在下一個年度使用 Azure 中特定數目的服務，以及事先支付該金額。 您所做的承諾可整年取用。 如果超過承諾金額，您可以事後支付超額部分。 所視承諾金額而定，您可享有 Azure 服務的折扣。

### <a name="grant-administrative-access-to-an-azure-subscription"></a>將系統管理存取權授予 Azure 訂用帳戶

RBAC 包含數個可供您用來指派權限的內建角色。 若要將使用者設定為 Azure 訂用帳戶的系統管理員，請在訂用帳戶範圍為其指派[擁有者](../../role-based-access-control/built-in-roles.md#owner)角色。 「擁有者」角色可授與使用者訂用帳戶中所有資源的完整存取權，包括將存取權委派給其他人的權限。

如需詳細資料，請參閱[使用 RBAC 和 Azure 入口網站來管理存取權](../../role-based-access-control/role-assignments-portal.md)。

### <a name="view-billing-information-in-the-azure-portal"></a>在 Azure 入口網站中檢視帳單資訊

使用 Azure 的重要元件是檢視帳單資訊的能力。 Azure 入口網站可供深入了解 Azure 帳單資訊。

如需詳細資訊，請參閱[如何下載您的 Azure 帳單發票和每日使用量資料](../../billing/billing-download-azure-invoice-daily-usage-date.md)。

### <a name="get-billing-information-from-billing-apis"></a>從計費 API 取得計費資訊

除了在入口網站中檢視計費資訊，您可以透過 Azure 計費 REST API 使用指令碼或程式來存取帳單資訊：

- 您可以使用 Azure 使用量 API 來擷取您的使用量資料。 您可以透過標記相關的 Azure 資源來微調使用量計費資訊。 例如，您可以使用部門名稱或專案名稱來標記資源群組中的每個資源，然後特別追蹤該標記的成本。

- 您可以使用 Azure Rate Card API 來列出所有可用的資源，以及每項資源的中繼資料和價格資訊。

如需詳細資訊，請參閱[深入瞭解 Microsoft Azure 資源耗用量](../../billing/billing-usage-rate-card-overview.md)。

### <a name="forecast-cost-with-the-pricing-calculator"></a>使用價格計算機預測成本

Azure 中每項服務的價格不同。 許多 Azure 服務都提供基本、標準和進階層。 通常，每一層都有數個價格和效能層級。 使用[線上價格計算機](http://azure.microsoft.com/pricing/calculator)，您可以建立價格估計值。 此計算機具備估計單一資源或資源群組成本的彈性。

## <a name="azure-resource-manager"></a>Azure Resource Manager

Azure Resource Manager 是 Azure 資源的部署、管理及組織機制。 使用 Resource Manager，您可以將許多個別資源一起放在資源群組中。

Resource Manager 還包含部署功能，可讓您自訂相關資源的部署和組態。 例如，使用 Resource Manager，即可部署將多個虛擬機器、一個負載平衡器和 SQL Database 當成一個單位處理的應用程式。 您可以使用 Resource Manager 範本來開發這些部署。

Resource Manager 會提供數個優點：

- 您可以以群組形式部署、管理及監視方案的所有資源，而不是個別處理這些資源。

- 您可以在整個方案週期重複部署方案，並確信您的資源會部署在一致的狀態中。

- 您可以透過宣告式範本而非指令碼來管理基礎結構。

- 您可以定義之間的相依性，使得以正確的順序部署資源。

- 因為 RBAC 會原生整合至管理平台，您可以將存取控制套用至資源群組中的所有服務。

- 您可以將標籤套用至資源，以邏輯方式組織訂用帳戶中的所有資源。

- 您可以檢視共用相同標籤之資源群組的成本，以釐清您的組織的計費方式。

### <a name="tips-for-creating-resource-groups"></a>建立資源群組的祕訣

當您正在進行資源群組的決策時，請考慮下列秘訣：

- 資源群組中的所有資源應具有相同的存留期。

- 您一次只能將一個資源指派給一個群組。

- 您可以隨時在資源群組中新增或移除資源。 每項資源必須屬於一個資源群組。 因此，如果您從一個群組中移除資源，您必須將該資源新增至另一個群組。

- 您可以隨時將大部分類型的資源移到不同的資源群組。

- 資源群組中的資源可位於不同的區域中。

- 您可以使用資源群組來控制其中資源的存取。

### <a name="building-resource-manager-templates"></a>建立 Resource Manager 範本

Resource Manager 範本會以宣告方式定義會部署到單一資源群組的資源和資源組態。 您可以使用 Resource Manager 範本來協調複雜的部署，而不需編寫額外的指令碼或手動設定。 在您開發範本之後，您可以多次進行部署 — 每次的結果都相同。

Resource Manager 範本包含四個區段：

- **參數**：這些是部署的輸入。 參數值可由人為或自動程序提供。 Windows VM 的系統管理員使用者名稱和密碼是一個參數範例。 指定的參數值會使用於整個部署。

- **變數**：這些用來保存整個部署使用的值。 不同於參數，部署期間不會提供變數值。 相反地，它是硬式編碼或動態產生。

- **資源**：範本的這個區段會定義所要部署的資源，例如虛擬機器、儲存體帳戶和虛擬網路。

- **輸出**：部署完成後，Resource Manager 可以傳回資料，例如動態產生的連接字串。

下列機制可供部署自動化：

- **功能**：您可以在 Resource Manager 範本中使用數個功能。 這些包括以下作業：將字串轉換為小寫、部署已定義資源的多個執行個體，並以動態方式傳回目標資源群組。 Resource Manager 功能可協助您建立動態部署。

- **資源相依性**：當您部署多個資源時，某些資源將會相依於其他資源。 若要加快部署，您可以使用相依性宣告，讓相依資源在其他資源之前部署。

- **範本連結**：在一個 Resource Manager 範本中，您可以連結至另一個範本。 這可讓部署分解成一組有目標、特定用途的範本。

您可以在任何文字編輯器中建立 Resource Manager 範本。 不過，Azure SDK for Visual Studio 包含可協助您的工具。 使用 Visual Studio，即可透過精靈將資源新增至範本，然後直接在 Visual Studio 中部署範本和進行偵錯。 如需詳細資訊，請參閱[編寫 Azure Resource Manager 範本](../../resource-group-authoring-templates.md)。

最後，您可以從 Azure 入口網站將現有資源群組轉換成可重複使用的範本。 如果您想要建立現有資源群組的可部署範本，或只是想要檢查基礎 JSON，這可能很有幫助。 若要匯出資源群組，請從資源群組的設定中選取 [自動化指令碼] 按鈕。

## <a name="security-of-azure-resources-rbac"></a>Azure 資源的安全性 (RBAC)

您可以授與指定範圍的使用者帳戶作業存取權：訂用帳戶、資源群組或個別資源。 這表示您可以將一組資源部署至資源群組，例如虛擬機器和所有相關資源，並將權限授與特定使用者或群組。 此方法讓存取僅限於屬於目標資源群組的資源。 您也可以授與對單一資源的存取權，例如虛擬機器或虛擬網路。

若要授與存取權，請將角色指派給使用者或使用者群組。 有許多預先定義的角色。 您也可以定義自己的自訂角色。

以下是一些範例 [Azure 內建角色](../../role-based-access-control/built-in-roles.md)：

- **擁有者**：擁有此角色的使用者可以管理所有項目，包括存取權。

- **讀取者**：擁有此角色的使用者可以讀取所有類型的資源 (祕密除外)，但無法進行變更。

- **虛擬機器參與者**：擁有此角色的使用者可以管理虛擬機器，但無法管理其連線的虛擬網路或 VHD 檔案所在的儲存體帳戶。

- **SQL DB 參與者**：擁有此角色的使用者可以管理 SQL 資料庫，但是無法管理其安全性相關原則。

- **SQL 安全性管理員**：擁有此角色的使用者可以管理 SQL Server 和資料庫的安全性相關原則。

- **儲存體帳戶參與者**：擁有此角色的使用者可以管理儲存體帳戶，但是無法管理對儲存體帳戶的存取權。

如需詳細資料，請參閱[使用 RBAC 和 Azure 入口網站來管理存取權](../../role-based-access-control/role-assignments-portal.md)。

## <a name="azure-virtual-machines"></a>Azure 虛擬機器

Azure 虛擬機器是 Azure 的其中一個中央 IaaS 服務。 Azure 虛擬機器支援在 Microsoft Azure 資料中心部署 Windows 或 Linux 虛擬機器。 使用 Azure 虛擬機器，您可以完全控制 VM 組態，且負責所有軟體安裝、設定和維護。

當您部署 Azure VM 時，您可以從 Azure Marketplace 中選取一個映像，也可以提供自己的一般化映像。 此映像用於套用作業系統和初始設定。 在部署期間，Resource Manager 會處理一些組態設定，例如指派電腦名稱、系統管理認證和網路組態。 您可以使用 Azure 虛擬機器擴充功能，進一步自動進行設定，例如軟體安裝、防毒設定以及監視解決方案。

您可以建立許多不同大小的虛擬機器。 虛擬機器的大小會決定資源配置，例如處理、記憶體和儲存體容量。 在某些情況下，特定功能 (例如已啟用 RDMA 的網路介面卡和 SSD 磁碟) 只適用於特定 VM 大小。 如需完整的 VM 大小和功能清單，請參閱適用於 [Windows](../../virtual-machines/windows/sizes.md) 和 [Linux](../../virtual-machines/linux/sizes.md) 的「Azure 虛擬機器大小」。

### <a name="use-cases"></a>使用案例

因為 Azure 虛擬機器提供組態的完全控制權，所以適合用於各種不符合 PaaS 模型的伺服器工作負載。 資料庫伺服器 (SQL Server、Oracle 或 MongoDB)、Windows Server Active Directory、Microsoft SharePoint 等伺服器工作負載便可能在 Microsoft Azure 平台上執行。 如有需要，您可以將這類工作負載從內部部署資料中心移到一或多個 Azure 區域，而不需進行大量重新設定。

### <a name="deployment-of-virtual-machines"></a>虛擬機器的部署

您可以使用 Azure 入口網站、使用自動化搭配 Azure PowerShell 模組，或使用自動化搭配跨平台 CLI 來部署 Azure 虛擬機器。

#### <a name="portal"></a>入口網站

使用 Azure 入口網站部署虛擬機器時，只需要有作用中的 Azure 訂用帳戶和網頁瀏覽器存取權。 您可以選取許多具有各種組態的不同作業系統映像。 在部署期間會設定所有的儲存體和網路需求。 如需詳細資訊，請參閱適用於 [Windows](../../virtual-machines/windows/quick-create-portal.md) 和 [Linux](../../virtual-machines/linux/quick-create-portal.md) 的「在 Azure 入口網站中建立虛擬機器」。

除了從 Azure 入口網站部署虛擬機器，您也可以從入口網站部署 Azure Resource Manager 範本。 這會部署和設定如範本中定義的所有資源。 如需詳細資訊，請參閱[使用 Resource Manager 範本與 Azure 入口網站來部署資源](../../azure-resource-manager/resource-group-template-deploy-portal.md)。

#### <a name="powershell"></a>PowerShell

使用 PowerShell 來部署 Azure 虛擬機器，可讓所有相關的虛擬機器資源 (包括儲存體和網路) 達到完全部署自動化。 如需詳細資訊，請參閱[使用 Resource Manager 和 PowerShell 建立 Windows VM](../../virtual-machines/windows/quick-create-powershell.md)。

除了個別部署 Azure 計算資源，您可以使用 Azure PowerShell 模組來部署 Azure Resource Manager 範本。 如需詳細資訊，請參閱[使用 Resource Manager 範本與 Azure PowerShell 來部署資源](../../azure-resource-manager/resource-group-template-deploy.md)。

#### <a name="command-line-interface-cli"></a>命令列介面 (CLI)

如同 PowerShell 模組，Azure 命令列介面提供部署自動化功能，並可使用於 Windows、OS X 或 Linux 系統。 當您使用 Azure CLI **vm quick-create** 命令時，會在部署所有相關的虛擬機器資源 (包括儲存體和網路) 和虛擬機器本身。 如需詳細資訊，請參閱 [使用 CLI 在 Azure 中建立 Linux VM](../../virtual-machines/linux/quick-create-cli.md)。

同樣地，您可以使用 Azure CLI 來部署 Azure Resource Manager 範本。 如需詳細資訊，請參閱[使用 Resource Manager 範本與 Azure CLI 來部署資源](../../azure-resource-manager/resource-group-template-deploy-cli.md)。

### <a name="access-and-security-for-virtual-machines"></a>虛擬機器的存取權與安全性

從網際網路存取虛擬機器時，需要使用公用 IP 位址來設定相關聯的網路介面或負載平衡器 (適用的話)。 公用 IP 位址包含將解析為虛擬機器或負載平衡器的 DNS 名稱。 如需詳細資訊，請參閱 [Azure 中的 IP 位址](../../virtual-network/virtual-network-ip-addresses-overview-arm.md)。

您可使用網路安全性群組 (NSG) 資源來管理透過公用 IP 位址的虛擬機器存取。 NSG 的作用如同防火牆，可在一組定義的連接埠上允許或拒絕跨越網路介面或子網路的流量。 例如，若要建立 Azure VM 的遠端桌面工作階段，您需要設定 NSG 才能允許連接埠 3389 的輸入流量。 如需詳細資訊，請參閱[使用 Azure 入口網站對 Azure 中的 VM 開啟連接埠](../../virtual-machines/windows/nsg-quickstart-portal.md)。

最後，如同任何電腦系統的管理，您應該使用安全性認證和軟體防火牆，在作業系統提供 Azure 虛擬機器的安全性。

## <a name="azure-storage"></a>Azure 儲存體

Azure 儲存體是 Microsoft 管理的服務，可提供持久、可擴充和備援儲存體。 您可以使用任何資源部署方法，將 Azure 儲存體帳戶新增為任何資源群組的資源。 Azure 包括四種儲存體類型：Blob 儲存體、檔案儲存體、表格儲存體和佇列儲存體。 部署儲存體帳戶時，可用的帳戶類型有兩種：一般用途和 blob 儲存體。 一般用途儲存體帳戶可讓您存取所有四個儲存體類型。 Blob 儲存體帳戶類似於一般用途的帳戶，但含有包括經常性和非經常性存取層的特製化 Blob。 如需 Blob 儲存體的詳細資訊，請參閱 [Azure Blob 儲存體](../../storage/blobs/storage-blob-storage-tiers.md)。

Azure 儲存體帳戶可以設定不同層級的備援性：

- **本機備援儲存體**確保在認定寫入成功之前，以同步方式為所有資料製作三個複本，進而提供高可用性。 這些複本會儲存在單一區域內的單一設備中。 這些複本會位於不同的容錯網域和升級網域中。 這表示即使保留資料的儲存體節點失敗，或已離線進行更新，資料仍可使用。

- **異地備援儲存體**會在主要區域中為資料製作三份同步副本，以提供高可用性，然後在配對區域中以非同步方式製作三個複本，以便進行災害復原。

- **讀取權限的異地備援儲存體**是異地備援儲存體，加上在次要區域中讀取資料的能力。 此功能適合用於部分災害復原。 如果主要區域發生問題，您可以將應用程式變更為具有配對區域的唯讀存取權。

### <a name="use-cases"></a>使用案例

每個儲存體類型都有不同的使用案例。

#### <a name="blob-storage"></a>Blob 儲存體

blob 這個字是「二進位大型物件的縮寫。 Blob 類似您在電腦上儲存的非結構化檔案。 Blob 儲存體可以儲存任何類型的文字或二進位資料，例如文件、媒體檔案或應用程式安裝程式。 Blob 儲存體也稱為物件儲存體。 Azure Blob 儲存體也可保留 Azure 虛擬機器資料磁碟。

Azure 儲存體支援三種 blob 類型：

- **區塊 blob**用來保存大小高達 195 GB (4 MB × 50,000 個區塊) 的一般檔案。 區塊 blob 的主要使用案例是從頭到尾都會被讀取之檔案的儲存體，例如網站的媒體檔案或映像檔案。 因為大於 64 MB 的檔案必須以小區塊的方式上傳，所以稱為區塊 blob。 這些區塊會接著合併 (或認可) 成最終 blob。

- **分頁 blob**用來保存隨機存取檔案 (大小上限為 1 TB)。 分頁 blob 主要作為 VHD (Azure 中的IaaS 計算服務) 的支援儲存體使用，而 VHD 可為 Azure 虛擬機器提供持久的磁碟。 因為它們提供 512 位元組頁面的隨機讀取/寫入存取權，所以稱為分頁 blob。

- **附加 Blob** 和區塊 Blob 相似，由區塊所組成，但已針對附加作業最佳化。 這類 Blob 經常用於將一或多個來源的資訊記錄至相同 blob。 例如，對於在多個 VM 上執行的應用程式，您可以將所有追蹤記錄寫入至相同的附加 blob。 單一附加 Blob 可以多達 195 GB。

如需詳細資訊，請參閱 [以 .NET 開始使用 Azure Blob 儲存體](../../storage/blobs/storage-dotnet-how-to-use-blobs.md)。

#### <a name="file-storage"></a>檔案儲存體

Azure 檔案儲存體是使用標準伺服器訊息區塊 (SMB) 通訊協定，在雲端中提供檔案共用功能的服務。 此服務支援 SMB 2.1 和 SMB 3.0。 使用 Azure 檔案儲存體時，您可以快速地將依賴檔案共用功能的應用程式移轉至 Azure，而不必浪費成本來重新撰寫程式。 在 Azure 虛擬機器、雲端服務或內部部署中執行的應用程式，可掛接雲端中的檔案共用。 類似於桌面應用程式掛接一般 SMB 共用的方式。 可同時掛接和存取檔案儲存體共用的應用程式元件數量沒有限制。

因為檔案儲存體共用是標準 SMB 檔案共用，所以 Azure 中執行的應用程式可透過檔案系統 I/O API 來存取共用中的資料。 因此，開發人員可使用現有的程式碼和技能來移轉現有的應用程式。 IT 專業人員在管理 Azure 應用程式時，可以使用 PowerShell Cmdlet 來建立、掛接和管理檔案儲存體共用。

如需詳細資訊，請參閱[在 Windows 上開始使用 Azure 檔案儲存體](../../storage/files/storage-how-to-use-files-windows.md)或[如何搭配 Linux 使用 Azure 檔案儲存體](../../storage/files/storage-how-to-use-files-linux.md)。

#### <a name="table-storage"></a>表格儲存體

Azure 表格儲存體是可將結構化的 NoSQL 資料儲存在雲端中的服務。 表格儲存體是具有無結構描述設計的索引鍵/屬性存放區。 由於表格儲存體並無結構描述，因此可輕易隨著應用程式發展需求改寫資料。 所有類型的應用程式都可以用快速且具成本效益的方式存取資料。 相較於類似資料量的傳統 SQL，資料表儲存體通常可大幅降低成本。

您可以使用表格儲存體來儲存具彈性的資料集，例如 Web 應用程式的使用者資料、通訊錄、裝置資訊，以及服務所需的任何其他中繼資料類型。 您可以在資料表中儲存任意數目的實體。 儲存體帳戶可包含任意數目的資料表，最高可達儲存體帳戶的容量限制。

如需詳細資訊，請參閱 [開始使用 Azure 資料表儲存體](../../cosmos-db/table-storage-how-to-use-dotnet.md)。

#### <a name="queue-storage"></a>佇列儲存體

Azure 佇列儲存體可提供應用程式元件之間的雲端傳訊。 設計擴充性的應用程式時，會經常分離應用程式元件，以便進行個別擴充。 佇列儲存體可針對應用程式元件間的通訊，提供非同步傳訊，無論應用程式元件是在雲端、桌面、內部部署伺服器或行動裝置上執行。 佇列儲存體也支援管理非同步工作並建置處理工作流程。

如需詳細資訊，請參閱 [開始使用 Azure 佇列儲存體](../../storage/queues/storage-dotnet-how-to-use-queues.md)。

### <a name="deploying-a-storage-account"></a>部署儲存體帳戶

有數個選項可以部署儲存體帳戶。

#### <a name="portal"></a>入口網站

使用 Azure 入口網站部署儲存體帳戶時，只需要有作用中的 Azure 訂用帳戶和網頁瀏覽器存取權。 您可以將新的儲存體帳戶部署到新的或現有的資源群組。 建立儲存體帳戶後，您可以使用入口網站來建立 blob 容器或檔案共用。 您可以用程式設計方式建立資料表和佇列儲存體實體。 如需詳細資訊，請參閱[建立儲存體帳戶](../../storage/common/storage-quickstart-create-account.md)。

除了從 Azure 入口網站部署儲存體帳戶，您也可以從入口網站部署 Azure Resource Manager 範本。 這會部署和設定如範本中定義的所有資源，包括任何儲存體帳戶。 如需詳細資訊，請參閱[使用 Resource Manager 範本與 Azure 入口網站來部署資源](../../azure-resource-manager/resource-group-template-deploy-portal.md)。

#### <a name="powershell"></a>PowerShell

使用 PowerShell 部署 Azure 儲存體帳戶，可讓儲存體帳戶達到完全部署自動化。 如需詳細資訊，請參閱[使用 Azure PowerShell 搭配 Azure 儲存體](../../storage/common/storage-powershell-guide-full.md)。

除了個別部署 Azure 資源，您可以使用 Azure PowerShell 模組來部署 Azure Resource Manager 範本。 如需詳細資訊，請參閱[使用 Resource Manager 範本與 Azure PowerShell 來部署資源](../../azure-resource-manager/resource-group-template-deploy.md)。

#### <a name="command-line-interface-cli"></a>命令列介面 (CLI)

如同 PowerShell 模組，Azure 命令列介面提供部署自動化功能，並可使用於 Windows、OS X 或 Linux 系統。 您可以使用 Azure CLI **storage account create** 命令來建立儲存體帳戶。 如需範例，請參閱[使用 Azure CLI 搭配 Azure 儲存體](../../storage/common/storage-azure-cli.md)。

同樣地，您可以使用 Azure CLI 來部署 Azure Resource Manager 範本。 如需詳細資訊，請參閱[使用 Resource Manager 範本與 Azure CLI 來部署資源](../../resource-group-template-deploy-cli.md)。

### <a name="access-and-security-for-azure-storage"></a>Azure 儲存體的存取權與安全性

Azure 儲存體可以不同的方式存取，包括透過 Azure 入口網站、在建立 VM 和作業期間，以及經由儲存體用戶端程式庫。

#### <a name="virtual-machine-disks"></a>虛擬機器磁碟

當您部署虛擬機器時，您也需要建立儲存體帳戶來保存虛擬機器作業系統磁碟和任何其他資料磁碟。 您可以選取現有的儲存體帳戶或建立一個新的。 因為 Blob 的大小上限是 1,024 GB，所以單一 VM 磁碟的大小上限為 1,023 GB。 若要設定較大的資料磁碟，您可以對虛擬機器呈現多個資料磁碟，並將它們集合為單一邏輯磁碟。 如需詳細資訊，請參閱適用於 [Windows](../../virtual-machines/windows/tutorial-manage-data-disk.md) 和 [Linux](../../virtual-machines/linux/tutorial-manage-disks.md) 的「管理 Azure 磁碟」。

#### <a name="storage-tools"></a>儲存體工具

Azure 儲存體帳戶可以透過許多不同的儲存體總管存取，例如 Visual Studio Cloud Explorer。 這些工具可讓您瀏覽儲存體帳戶和資料。 如需詳細資訊和可用儲存體總管清單，請參閱 [Azure 儲存體用戶端工具](../../storage/common/storage-explorers.md)。

#### <a name="storage-api"></a>儲存體 API

任何可提出 HTTP/HTTPS 要求的語言皆可存取儲存體資源。 另外，Azure 儲存體還提供了幾種熱門語言的程式設計程式庫。 這些程式庫可透過處理詳細資料 (例如同步和非同步叫用、進行批次作業、例外狀況管理和自動重試) 來簡化 Azure 儲存體使用方式。 如需詳細資訊，請參閱 [Azure 儲存體服務 REST API 參考](/rest/api/storageservices/Azure-Storage-Services-REST-API-Reference)。

#### <a name="storage-access-keys"></a>儲存體存取金鑰

每個儲存體帳戶有兩種驗證金鑰：主要和次要。 任一金鑰均可用於儲存體存取作業。 這些儲存體金鑰用來協助保護儲存體帳戶，而且是以程式設計方式存取資料的必要項目。 有兩個金鑰可允許為了增強安全性而偶爾變換金鑰。 務必將金鑰放在安全的地方，因為其擁有權 (連同帳戶名稱) 允許無限制存取儲存體帳戶中的任何資料。

#### <a name="shared-access-signatures"></a>共用存取簽章

如果您需要允許使用者對儲存體資源進行控管存取，則您可以建立共用存取簽章。 共用存取簽章是指可附加在 URL 後面的權杖，可提供儲存體資源的委派存取。 在權杖有效的期限內，擁有權杖的任何人都可以存取它在指定權限中所指向的資源。 如需詳細資訊，請參閱[使用共用存取簽章](../../storage/common/storage-dotnet-shared-access-signature-part-1.md)。

## <a name="azure-virtual-network"></a>Azure 虛擬網路

需要虛擬網路以支援虛擬機器之間的通訊。 您可以定義子網路、自訂 IP 位址、DNS 設定、安全性篩選，以及負載平衡。 Azure 支援不同的使用案例：僅限雲端的網路或混合式虛擬網路。

### <a name="cloud-only-virtual-networks"></a>僅限雲端虛擬網路

根據預設，Azure 虛擬網路僅供 Azure 中儲存的資源存取。 連線至相同虛擬網路的資源可以彼此通訊。 您可以讓虛擬機器網路介面和負載平衡器與公用 IP 位址產生關聯，以便透過網際網路存取虛擬機器。 您可以使用網路安全性群組，協助安全地存取公開資源。

![2 層式 Web 應用程式的 Azure 虛擬網路](https://docs.microsoft.com/azure/load-balancer/media/load-balancer-internal-overview/ic744147.png)

### <a name="hybrid-virtual-networks"></a>混合式虛擬網路

您可以使用 ExpressRoute 或站對站 VPN 連線，將內部部署網路連線到 Azure 虛擬網路。 在這個設定中，Azure 虛擬網路本質上是您的內部部署網路中具備雲端功能的延伸模組。
![使用 VPN 的混合式虛擬網路](https://docs.microsoft.com/azure/architecture/reference-architectures/_images/blueprints/hybrid-network-vpn.png)

由於 Azure 虛擬網路會連線到您的內部部署網路，所以跨單位虛擬網路必須使用您組織所用位址空間的唯一部分。 與為不同的公司位置指派特定 IP 子網路相同，Azure 會隨著您擴充網路而成為另一個位置。
有數個選項可供部署虛擬網路。

- [入口網站](../..//virtual-network/quick-create-portal.md)

- [PowerShell](../../virtual-network/quick-create-powershell.md)

- [命令列介面 (CLI)](../../virtual-network/quick-create-cli.md)

- Azure Resource Manager 範本

>**使用時機**： 每當您在 Azure 中使用 VM 時，您就會使用虛擬網路。 這可允許將 VM 分割成公開面向和類似私人子網路的內部部署資料中心。

>**開始使用**：使用 Azure 入口網站部署 Azure 虛擬網路時，只需要有作用中的 Azure 訂用帳戶和網頁瀏覽器存取權。 您可以將新的虛擬網路部署到新的或現有的資源群組。 當您要從入口網站建立新的虛擬機器時，您可以選取現有的虛擬網路或建立一個新的。 開始使用並[使用 Azure 入口網站建立虛擬網路](../../virtual-network/quick-create-portal.md)。

### <a name="access-and-security-for-virtual-networks"></a>虛擬網路的存取權與安全性

您可以使用網路安全性群組，協助保護 Azure 虛擬網路。 NSG 包含存取控制清單 (ACL) 規則的清單，可允許或拒絕虛擬網路中 VM 執行個體的網路流量。 您可以讓 NSG 與子網路或該子網路內的個別 VM 執行個體產生關聯。 當您讓 NSG 與子網路產生關聯時，ACL 規則便會套用至該子網路中的所有 VM 執行個體。 此外，讓 NSG 直接與該 VM 產生關聯，即可進一步限制個別 VM 的流量。 如需詳細資訊，請參閱[使用網路安全性群組來篩選網路流量](../../virtual-network/security-overview.md)。

## <a name="next-steps"></a>後續步驟

- [建立 Windows VM](../../virtual-machines/windows/quick-create-portal.md)
- [建立 Linux VM](../../virtual-machines/linux/quick-create-portal.md)
