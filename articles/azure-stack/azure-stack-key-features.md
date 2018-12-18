---
title: Azure Stack 的主要功能與概念 | Microsoft Docs
description: 深入了解 Azure Stack 中的主要功能與概念
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: 09ca32b7-0e81-4a27-a6cc-0ba90441d097
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/10/2018
ms.author: jeffgilb
ms.reviewer: ''
ms.openlocfilehash: 851530910c702d388cd4dc8607bf09ecb5fa44e0
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34198468"
---
# <a name="key-features-and-concepts-in-azure-stack"></a>Azure Stack 的主要功能與概念
如果您是 Microsoft Azure Stack 的新手，這些字詞與功能描述可能相當實用。

## <a name="personas"></a>角色
Microsoft Azure Stack 有兩種不同的使用者，也就是雲端操作員 (提供者) 與租用戶 (消費者)。

* **雲端操作員**可以設定 Azure Stack 及管理產品、方案、服務、配額及定價，為他們的租用戶提供資源。  雲端操作員也管理容量並會回應警示。  
* **租用戶** (也稱為使用者) 將會使用雲端系統管理員所提供的服務。 租用戶可以佈建、監視及管理他們所訂閱的服務，例如 Web Apps、儲存體以及虛擬機器。

## <a name="portal"></a>入口網站
與 Microsoft Azure Stack 互動的主要方法是系統管理員入口網站、使用者入口網站，以及 PowerShell。

每個 Azure Stack 入口網站皆由 Azure Resource Manager 的個別執行個體支援。  雲端操作員會使用系統管理員入口網站來管理 Azure Stack，以及執行一些作業，例如建立租用戶產品。  使用者入口網站 (也稱為租用戶入口網站) 提供使用雲端資源的自助式服務體驗，像是虛擬機器、儲存體帳戶，以及 Web Apps。 如需詳細資訊，請參閱[使用 Azure Stack 中系統管理員和使用者的入口網站](azure-stack-manage-portals.md)。

## <a name="identity"></a>身分識別 
Azure Stack 使用 Azure Active Directory (AAD) 或 Active Directory 同盟服務 (AD FS) 作為識別提供者。  

### <a name="azure-active-directory"></a>Azure Active Directory
Azure Active Directory 是 Microsoft 的雲端式、多租用識別提供者。  大部分的混合式情節使用 Azure Active Directory 作為身分識別存放區。

### <a name="active-directory-federation-services"></a>Active Directory Federation Services
您可以選擇使用 Active Directory 同盟服務 (AD FS) 作為中斷連線的 Azure Stack 部署。  Azure Stack、資源提供者和其他應用程式使用 AD FS 的方式，與使用 Azure Active Directory 的方式差不多。 Azure Stack 包含自己的 AD FS 和 Active Directory 執行個體，以及 Active Directory 圖形 API。 Azure Stack 開發套件支援下列 AD FS 情節：

- 使用 AD FS 登入部署。
- 使用金鑰保存庫中的祕密建立虛擬機器
- 建立用於儲存/存取機密資料的保存庫
- 建立訂用帳戶中的自訂 RBAC 角色
- 指派使用者 RBAC 角色的資源
- 建立透過 Azure PowerShell 的全系統 RBAC 角色
- 透過 Azure PowerShell 的使用者身分登入
- 建立服務主體並用其登入 Azure PowerShell


## <a name="regions-services-plans-offers-and-subscriptions"></a>區域、服務、方案、產品以及訂用帳戶
在 Azure Stack 中，能透過區域、訂用帳戶、產品與各種方案，為租用戶提供服務。 租用戶可以訂閱多項優惠。 優惠可以有一個或多個方案，而方案則可有一或多項服務。

![](media/azure-stack-key-features/image4.png)

租用戶各項優惠 (各有不同方案與服務) 的訂用帳戶階層範例。

### <a name="regions"></a>區域
Azure Stack 區域是級別和管理的基本元素。 組織可能會有多重區域，而每個區域皆有可用資源。 區域可能也會有不同服務的產品可供使用。 在 Azure Stack 開發套件中，只支援單一區域且會自動命名為「local」。

### <a name="services"></a>服務
提供者可透過 Microsoft Azure Stack 提供各種不同的服務與應用程式，例如虛擬機器、SQL Server 資料庫、SharePoint、Exchange 及其他項目。

### <a name="plans"></a>方案
方案結合一或多項服務。 身為提供者的您可以為租用戶製作方案。 租用戶接著即可訂閱各項優惠，使用其內含的方案與服務。

每項加入方案中的服務，皆可設定配額以協助您管理雲端容量。 配額可內含限制，像是 VM、RAM 與 CPU 的限制，且會以每位使用者的訂用帳戶為單位加以套用。 配額可能會依位置而有所不同。 例如，包含來自區域 A 的計算服務之方案，可能會有兩部虛擬機器、4 GB RAM 與 10 個 CPU 核心的配額。

建立產品時，服務系統管理員可以內含**基本方案**。 當租用戶訂閱該優惠時，預設會包含這些基本方案。 當使用者訂閱 (且已建立訂用帳戶) 之後，該使用者即可存取基本方案 (具備相對應的配額) 中所指定的所有資源提供者。

服務系統管理員也可以在優惠中內含 **加購方案** 。 訂用帳戶中預設不會包含加購方案。 產品中提供的加購方案屬於額外的方案 (配額)，擁有者可將其加入其訂用帳戶中。

### <a name="offers"></a>優惠
優惠是一組提供者供租用戶購買 (訂閱) 的一或多項方案。 例如，產品 Alpha 可以包含方案 A (包含一組計算服務) 與方案 B (包含一組儲存體與網路服務)。

優惠配備一組基本方案，而服務系統管理員可以建立租用戶可加入其訂用帳戶的加購方案。

### <a name="subscriptions"></a>訂用帳戶
訂用帳戶是租用戶購買優惠的方式。 訂用帳戶是租用戶與優惠的組合。 租用戶可以有多項優惠的訂用帳戶。 每個訂用帳戶僅適用於一項優惠。 租用戶的訂用帳戶會決定他們可以存取哪些方案/服務。

訂用帳戶能協助提供者整理和存取雲端資源與服務。

對於系統管理員，在部署期間則會建立「預設提供者訂閱」。 此訂用帳戶可用來管理 Azure Stack、部署其他資源提供者，以及為租用戶建立方案與產品。 此訂用帳戶不應用於執行客戶工作負載和應用程式。 從版本 1804 開始，有兩個額外的訂用帳戶可補充預設提供者訂用帳戶：計量訂用帳戶和取用訂用帳戶。 這些新增項目有助於區隔核心基礎結構、額外的資源提供者及工作負載的管理。  

## <a name="azure-resource-manager"></a>Azure Resource Manager
透過使用 Azure Resource Manager，您能夠以範本為基礎、宣告式模型來使用基礎結構資源。   其提供單一介面，您可用來部署和管理解決方案的元件。 如需完整的資訊與指引，請參閱 [Azure Resource Manager 概觀](../azure-resource-manager/resource-group-overview.md)。

### <a name="resource-groups"></a>資源群組
資源群組是資源、服務與應用程式的集合，而且每個資源各有類型，例如虛擬機器、虛擬網路、公用 IP、儲存體帳戶與網站。 每個資源都必須位於資源群組中，而資源群組有助於整理本機資源，例如依工作負載或位置。  在 Microsoft Azure Stack，像是方案與優惠等資源，也會於資源群組中進行管理。

不同於 [Azure](../azure-resource-manager/resource-group-move-resources.md)，您無法在資源群組間移動資源。 當您在 Azure Stack 管理入口網站中檢視資源或資源群組的屬性時，[移動] 按鈕會呈現灰色且無法使用。 
 
### <a name="azure-resource-manager-templates"></a>Azure 資源管理員範本
利用 Azure Resource Manager，您可以建立可定義應用程式之部署和設定的範本 (以 JSON 格式)。 此範本就是所謂的 Azure 資源管理員範本，可提供定義部署的宣告方式。 藉由使用範本，您可以在整個應用程式週期重複部署應用程式，並確信您的資源會部署在一致的狀態中。

## <a name="resource-providers-rps"></a>資源提供者 (RP)
資源提供者即是為 Azure 形式 IaaS 與 PaaS 服務形成基礎的 Web 服務。 Azure Resource Manager 仰賴不同的資源提供者來存取服務。

有四個基本的資源提供者：網路、儲存體、計算和 KeyVault。 每個這些資源提供者，皆可協助您設定及控制其各自的資源。 服務管理員也可以加入新的自訂資源提供者。

### <a name="compute-rp"></a>計算 RP
Azure Stack 的租用戶可利用計算資源提供者 (CRP)，建立自己的虛擬機器。 CRP 也有功能可建立虛擬機器，以及虛擬機器擴充功能。 虛擬機器擴充功能服務有助於為 Windows 與 Linux 虛擬機器提供 IaaS 功能。  例如，您可以使用 CRP 來佈建 Linux 虛擬機器，並在部署期間執行 Bash 指令碼來設定虛擬機器。

### <a name="network-rp"></a>網路 RP
網路資源提供者 (NRP) 能為私人雲端提供一系列的軟體定義網路功能 (SDN) 以及網路函式虛擬化 (NFV) 功能。  您可以使用 NRP 建立資源，像是軟體負載平衡器、公用 IP、網路安全性群組，以及虛擬網路。

### <a name="storage-rp"></a>儲存體 RP
儲存體 RP 能提供四種與 Azure 一致的儲存體服務：BLOB、資料表、佇列與帳戶管理。 它也提供儲存體雲端管理服務，來輔助與 Azure 一致之儲存體服務的服務提供者管理。 Azure 儲存體能為儲存及擷取大量非結構化資料提供彈性，例如 Azure Blob 的文件與媒體檔案，以及具備 Azure 資料表的結構化 NoSQL 資料。 如需有關 Azure 儲存體的詳細資訊，請參閱 [Microsoft Azure 儲存體簡介](../storage/common/storage-introduction.md)。

#### <a name="blob-storage"></a>Blob 儲存體
Blob 儲存體可儲存任何檔案集。 Blob 可以是任何類型的文字或二進位資料，例如文件、媒體檔案或應用程式安裝程式。 資料表儲存體可儲存結構化的資料集。 資料表儲存體屬於 NoSQL 索引鍵屬性資料儲存，可允許快速開發和迅速存取大量資料。 佇列儲存體針對工作流程處理及雲端服務元件間的通訊，提供可靠的訊息服務。

每個 Blob 會在容器下形成組織。 容器也提供對物件群組指派安全原則的實用方式。 儲存體帳戶可包含任意數目的容器，而容器可包含任意數目的 Blob，儲存體帳戶的容量限制高達 500 TB。 Blob 儲存體提供三種類型的 Blob，包括區塊 Blob、附加 Blob 及分頁 Blob (磁碟)。 區塊 Blob 已針對串流和儲存雲端物件進行最佳化，是儲存文件、媒體檔案、備份等的不錯選擇。附加 Blob 和區塊 Blob 類似，但已針對附加作業最佳化。 附加 Blob 只能透過在結尾新增新的區塊來更新。 對於如記錄等僅需要將新資料寫入 Blob 結尾的情況，附加 Blob 是不錯的選擇。 頁面 Blob 已針對顯示 IaaS 磁碟與支援隨機寫入進行最佳化，而且可能可以高達 1 TB。 Azure 虛擬機器網路連結的 IaaS 磁碟是以頁面 Blob 方式儲存的 VHD。

#### <a name="table-storage"></a>表格儲存體
表格儲存體屬於 Microsoft 的 NoSQL 索引鍵/屬性存放區，其設計成沒有結構描述，使其有別於傳統的關聯式資料庫。 因為資料存放區沒有結構描述，所以可輕易隨著應用程式發展需求來更動資料。 資料表儲存體非常容易使用，方便開發人員可以快速建立應用程式。 資料表儲存體屬於索引鍵屬性儲存，代表資料表中的每個值會與具型別的屬性名稱一起儲存。 屬性名稱可用來篩選與指定選取條件。 一組屬性及其值就構成一個實體。 因為表格儲存體沒有結構描述，因此相同資料表中的兩個實體可包含不同的屬性集合，且這些屬性可以是不同類型。 您可以使用資料表儲存體來儲存具彈性的資料集，例如 Web 應用程式的使用者資料、通訊錄、裝置資訊，以及服務所需的任何其他中繼資料類型。 您可以在資料表中儲存任意數目的實體，且儲存體帳戶可包含任意數目的資料表，最高可達儲存體帳戶的容量限制。

#### <a name="queue-storage"></a>佇列儲存體
Azure 佇列儲存體可提供應用程式元件之間的雲端傳訊。 設計擴充性的應用程式時，會經常分離應用程式元件，以便進行個別擴充。 佇列儲存體可針對應用程式元件間的通訊，提供非同步傳訊，無論應用程式元件是在雲端、桌面、內部部署伺服器或行動裝置上執行。 佇列儲存體也支援管理非同步工作並建置處理工作流程。

### <a name="keyvault"></a>KeyVault
KeyVault RP 提供管理和稽核的祕密，例如密碼和憑證。 例如，租用戶可以使用 KeyVault RP 來在虛擬機器部署期間，提供系統管理員密碼或金鑰。

## <a name="high-availability-for-azure-stack"></a>Azure Stack 的高可用性
*適用於：Azure Stack 1802 或更新版本*

為了達到 Azure 中多 VM 生產環境的高可用性，會將 VM 放在可用性設定組中，此設定組會將 VM 分散在多個容錯網域和更新網域中。 如此一來，[部署在可用性設定組中的 VM](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-availability-sets) 便實體上在個別的伺服器機架上彼此隔離，而可提供失敗復原能力，如下圖所示：

  ![Azure Stack 高可用性](media/azure-stack-key-features/high-availability.png)

### <a name="availability-sets-in-azure-stack"></a>Azure Stack 中的可用性設定組
雖然 Azure Stack 的基礎結構已經具備失敗復原能力，但在發生硬體故障時，基礎技術 (容錯移轉叢集) 仍然會造成受影響實體伺服器上的 VM 產生一些停機時間。 Azure Stack 支援的可用性設定組最多可以有三個容錯網域 (與 Azure 一致)。

- **容錯網域**。 系統會將放在可用性設定組中的 VM 儘可能平均分散到多個容錯網域 (Azure Stack 節點)，讓這些 VM 在實體上彼此隔離。 當發生硬體故障時，失敗容錯網域中的 VM 會在其他容錯網域中重新啟動，但可能的話，會放在與相同可用性設定組中其他 VM 不同的容錯網域中。 當硬體回到線上時，系統會將 VM 重新平衡以保持高可用性。 
 
- **更新網域**。 更新網域是另一個可在可用性設定組中提供高可用性的 Azure 概念。 更新網域是可以同時進行維護的基礎硬體邏輯群組。 位於相同更新網域中的 VM 會在預定進行的維護期間一起重新啟動。 當租用戶在可用性設定組內建立 VM 時，Azure 平台會自動將 VM 分散到這些更新網域中。 在 Azure Stack 中，會先將 VM 即時移轉至叢集內的各個其他線上主機，然後才更新 VM 的基礎主機。 由於在主機更新期間並不會導致租用戶停機，因此 Azure Stack 上更新網域功能的存在只是為了與 Azure 的範本相容。 

### <a name="upgrade-scenarios"></a>升級案例 
可用性設定組中的 VM 若是在 Azure Stack 1802 版之前建立的，會被賦予預設的容錯和更新網域數目 (分別為 1 和 1)。 為了讓在這些預先存在之可用性設定組中的 VM 達到高可用性，您必須先刪除現有的 VM，然後將它們重新部署至具有正確容錯和更新網域計數的新可用性設定組中，如[變更 Windows VM 的可用性設定組](https://docs.microsoft.com/azure/virtual-machines/windows/change-availability-set)所述。 

針對虛擬機器擴展集，會在內部建立具有預設容錯網域和更新網域計數 (分別為 3 和 5) 的可用性設定組。 任何虛擬機器擴展集只要是在 1802 更新之前建立的，都會放在具有預設容錯和更新網域計數 (分別為 1 和 1) 的可用性設定組中。 若要更新這些虛擬機器擴展集以達到較新的分配，請依據 1802 更新之前便已存在的執行個體數目，將虛擬機器擴展集向外延展，然後刪除較舊的虛擬機器擴展集執行個體。 

## <a name="role-based-access-control-rbac"></a>角色型存取控制 (RBAC)
您可使用 RBAC 在訂用帳戶、資源群組或各個資源層級，為授權的使用者、群組與服務指派角色，即可為其授與系統存取權。 每個角色皆會定義使用者、群組或服務對於 Microsoft Azure Stack 資源所具有的存取層級。

Azure RBAC 有適用於所有資源類型的三個基本角色：擁有者、參與者和讀者。 擁有者具有所有資源的完整存取權，包括將存取權委派給其他人的權限。 參與者可以建立和管理所有類型的 Azure 資源，但是不能將存取權授與其他人。 讀者只能檢視現有的 Azure 資源。 Azure 中其餘的 RBAC 角色可以管理特定 Azure 資源。 例如，虛擬機器參與者角色可以建立和管理虛擬機器，但是無法管理虛擬網路或虛擬機器所連接的子網路。

## <a name="usage-data"></a>使用狀況資料
Microsoft Azure Stack 會從所有資源提供者收集彙總的使用方式資料，並將其傳輸至 Azure 以由 Azure 商務進行處理。 透過 REST API，您可以檢視 Azure Stack 上收集的使用方式資料。 同時有與 Azure 一致的租用戶 API 和提供者，以及委派的提供者 API，以取得所有租用戶訂用帳戶的使用量資料。 此資料可用於與外部工具或收費或退款服務相整合。 一旦使用方式經由 Azure 商務處理，其就可以在 Azure 計費入口網站中檢視。

## <a name="in-development-build-of-azure-stack-development-kit"></a>正在開發的 Azure Stack 開發套件組建
正在開發組建可以讓早期採用者評估 Azure Stack 開發套件的最新版本。 這是以最新主要版本為基礎累加的組建。 雖然主要版本將會每幾個月繼續發佈，正在開發組建會間歇性地在主要版本之間發佈。

正在開發組建可提供下列優點：
- 錯誤修正
- 新功能
- 其他改進功能

## <a name="next-steps"></a>後續步驟
[評估 Azure Stack 開發套件](azure-stack-deploy-overview.md)

