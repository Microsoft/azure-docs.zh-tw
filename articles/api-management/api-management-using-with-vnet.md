---
title: 如何將 Azure API 管理與虛擬網路搭配使用
description: 了解如何在「Azure API 管理」中設定虛擬網路連線，然後透過它存取 Web 服務。
services: api-management
documentationcenter: ''
author: vlvinogr
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/05/2017
ms.author: apimpm
ms.openlocfilehash: deba3ad8a283b111dc94a5361f3fa4e73d95c0b8
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39187378"
---
# <a name="how-to-use-azure-api-management-with-virtual-networks"></a>如何將 Azure API 管理與虛擬網路搭配使用
「Azure 虛擬網路」(VNET) 可讓您將任何 Azure 資源，放在您控制存取權的非網際網路可路由網路中。 然後，可以使用各種 VPN 技術，將這些網路連線到您的內部部署網路。 若要深入了解「Azure 虛擬網路」，請從以下資訊著手：[Azure 虛擬網路概觀](../virtual-network/virtual-networks-overview.md)。

Azure API 管理可以部署在虛擬網路 (VNET) 內，因此它可以存取網路內的後端服務。 開發人員入口網站與 API 閘道，可設定為從網際網路存取或只從虛擬網路內存取。

> [!NOTE]
> Azure API 管理支援傳統和 Azure Resource Manager Vnet。
>

## <a name="prerequisites"></a>必要條件

若要執行本文所述的步驟，您必須具有：

+ 有效的 Azure 訂用帳戶。

    [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

+ APIM 執行個體。 如需詳細資訊，請參閱[建立 Azure API 管理執行個體](get-started-create-service-instance.md)。
+ VNET 連線僅適用於「進階」和「開發人員」層。 請遵循[升級和調整](upgrade-and-scale.md#upgrade-and-scale)主題中的指示，切換至上述其中一層。

## <a name="enable-vpn"> </a>啟用 VNET 連線

### <a name="enable-vnet-connectivity-using-the-azure-portal"></a>使用 Azure 入口網站啟用 VNET 連線能力

1. 在 [Azure 入口網站](https://portal.azure.com/)中，瀏覽至您的 APIM 執行個體。
2. 選取 [虛擬網路]。
3. 將 API 管理執行個體設定為在虛擬網路內部署。

    ![API 管理的虛擬網路功能表][api-management-using-vnet-menu]
4. 選取所需的存取類型：

    * **外部**：可透過外部負載平衡器，從公用網際網路存取「API 管理」閘道和開發人員入口網站。 閘道可以存取虛擬網路內的資源。

    ![公用對等互連][api-management-vnet-public]

    * **內部**：只能透過內部負載平衡器，從虛擬網路內存取「API 管理」閘道和開發人員入口網站。 閘道可以存取虛擬網路內的資源。

    ![私人對等互連][api-management-vnet-private]`

    您現在會看見佈建 API 管理服務所在的所有區域的清單。 為每個區域選取 VNET 和子網路。 此清單中會同時填入傳統和 Resource Manager 虛擬網路，這些您要設定之區域中所設定 Azure 訂用帳戶可用的虛擬網路。

    > [!NOTE]
    > 上圖中的**服務端點**包括閘道/Proxy、Azure 入口網站、開發人員入口網站、GIT 及直接管理端點。
    > 上圖中的「管理端點」是裝載在服務上以透過 Azure 入口網站和 Powershell 來管理組態的端點。
    > 此外，請注意，即使此圖顯示其各種端點的「IP 位址」，「API 管理」服務仍然「只」會在其已設定的「主機名稱」上回應。

    > [!IMPORTANT]
    > 將 Azure API 管理執行個體部署至 Resource Manager VNET 時，服務必須在除了 Azure API 管理執行個體之外不包含其他資源的專用子網路中。 如果嘗試將 Azure API 管理執行個體部署到含有其他資源的 Resource Manager VNET 子網路，則部署將會失敗。
    >

    ![選取 VPN][api-management-setup-vpn-select]

5. 按一下畫面頂端的 [儲存]。

> [!NOTE]
> 「API 管理」執行個體的 VIP 位址在每次啟用或停用 VNET 時都會變更。
> 將 API 管理中**外部**移動至**內部**或反之時，也會變更的 VIP 位址
>

> [!IMPORTANT]
> 如果您將「API 管理」從 VNET 中移除或變更其部署所在的 VNET，則先前使用的 VNET 將保持鎖定狀態最長達兩個小時。 在這段期間，將無法刪除該 VNET 或在其中部署新的資源。

## <a name="enable-vnet-powershell"> </a>使用 PowerShell cmdlet 來啟用 VNET 連線
您也可以使用 PowerShell cmdlet 啟用 VNET 連線能力

* **在 VNET 內建立 API 管理服務**：使用 cmdlet [New-AzureRmApiManagement](/powershell/module/azurerm.apimanagement/new-azurermapimanagement) 在 VNET 內建立 Azure API 管理服務。

* **在 VNET 內部署現有 API 管理服務**：使用 cmdlet [Update-AzureRmApiManagementDeployment](/powershell/module/azurerm.apimanagement/update-azurermapimanagementdeployment) 移動虛擬網路內的現有 Azure API 管理服務。

## <a name="connect-vnet"> </a>連接到裝載於虛擬網路內的 Web 服務
在您的「API 管理」服務連接到 VNET 之後，存取 VNET 內的後端服務與存取公用服務便沒有差別。 只要在建立新 API 或編輯現有 API 時，於 [Web 服務 URL] 欄位中輸入您 Web 服務的本機 IP 位址或主機名稱 (如果為 VNET 設定了 DNS 伺服器) 即可。

![透過 VPN 加入 API][api-management-setup-vpn-add-api]

## <a name="network-configuration-issues"> </a>常見的網路組態問題
以下是將 API 管理服務部署到虛擬網路時可能發生的常見錯誤設定問題清單。

* **自訂 DNS 伺服器安裝**：API 管理服務相依於數個 Azure 服務。 當「API 管理」是裝載於具有自訂 DNS 伺服器的 VNET 中時，它必須要解析這些 Azure 服務的主機名稱。 請遵循 [這份](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server) 有關自訂 DNS 設定的指引。 請參閱下方的連接埠資料表和參考的其他網路需求。

> [!IMPORTANT]
> 如果您打算針對 VNET 使用「自訂 DNS 伺服器」，在將 API 管理服務部署到該伺服器**之前**，應該先將該伺服器設定妥當。 否則，每次變更 DNS 伺服器時，您都必須執行[套用網路設定作業](https://docs.microsoft.com/rest/api/apimanagement/ApiManagementService/ApplyNetworkConfigurationUpdates)來更新 API 管理服務

* **API 管理所需的連接埠**︰使用[網路安全性群組][Network Security Group]可以控制到 API 管理部署於其中的子網路之輸入和輸出流量。 如果這些連接埠中有任何一個無法使用，「API 管理」可能就無法正常運作而可能變成無法存取。 搭配 VNET 使用 API 管理時，封鎖這其中一或多個連接埠是另一個常見的錯誤組態問題。

當 API 管理服務執行個體裝載於 VNET 時，會使用下表中的連接埠。

| 來源 / 目的地連接埠 | 方向 | 傳輸通訊協定 | 來源 / 目的地 | 目的 (*) | 虛擬網路類型 |
| --- | --- | --- | --- | --- | --- |
| * / 80, 443 |輸入 |TCP |INTERNET / VIRTUAL_NETWORK|與 API 管理的用戶端通訊|外部 |
| * / 3443 |輸入 |TCP |INTERNET / VIRTUAL_NETWORK|Azure 入口網站和 PowerShell 的管理端點 |內部 |
| * / 80, 443 |輸出 |TCP |VIRTUAL_NETWORK / INTERNET|與「Azure 儲存體」、「Azure 服務匯流排」及 Azure Active Directory 的**相依性** (如果適用)。|外部和內部 |
| * / 1433 |輸出 |TCP |VIRTUAL_NETWORK / SQL|**存取 Azure SQL 端點** |外部和內部 |
| * / 5672 |輸出 |TCP |VIRTUAL_NETWORK / INTERNET|「記錄到事件中樞」原則和監視代理程式的相依性 |外部和內部 |
| * / 445 |輸出 |TCP |VIRTUAL_NETWORK / INTERNET|與「適用於 GIT 的 Azure 檔案共用」的相依性 |外部和內部 |
| * / 1886 |輸出 |TCP |VIRTUAL_NETWORK / INTERNET|將健康情況狀態發佈至 [資源健康狀態] 時所需 |外部和內部 |
| * / 25028 |輸出 |TCP |VIRTUAL_NETWORK / INTERNET|連線到 SMTP 轉送以便傳送電子郵件 |外部和內部 |
| * / 6381 - 6383 |輸入和輸出 |TCP |VIRTUAL_NETWORK / VIRTUAL_NETWORK|存取 RoleInstances 之間的 Redis 快取執行個體 |外部和內部 |
| * / * | 輸入 |TCP |AZURE_LOAD_BALANCER / VIRTUAL_NETWORK| Azure 基礎結構負載平衡器 |外部和內部 |

>[!IMPORTANT]
> 要成功部署 API 管理服務，就必須有以**粗體**表示其「目的」的連接埠。 不過，封鎖其他連接埠將會降低使用和監視執行中服務的能力。

* **SSL 功能**︰若要啟用 SSL 憑證鏈結建立和驗證，API 管理服務需要 ocsp.msocsp.com、mscrl.microsoft.com 和 crl.microsoft.com 的輸出網路連線。 如果您上傳至 API 管理的任何憑證包含 CA 根的完整鏈結，則不需要此相依性。

* **DNS 存取**：需要有連接埠 53 的輸出存取，才能與 DNS 伺服器通訊。 如果 VPN 閘道的另一端有自訂 DNS 伺服器存在，則必須可從裝載 API 管理的子網路連接該 DNS 伺服器。

* **計量和健康情況監視**︰對 Azure 監視端點 (解析為屬於下列網域) 的輸出網路連線能力︰ 

    | Azure 環境 | 端點 |
    | --- | --- |
    | Azure 公用 | <ul><li>prod.warmpath.msftcloudes.com</li><li>shoebox2.metrics.nsatc.net</li><li>prod3.metrics.nsatc.net</li><li>prod3-black.prod3.metrics.nsatc.net</li><li>prod3-red.prod3.metrics.nsatc.net</li></ul> |
    | Azure Government | <ul><li>fairfax.warmpath.usgovcloudapi.net</li><li>shoebox2.metrics.nsatc.net</li><li>prod3.metrics.nsatc.net</li></ul> |
    | Azure China | <ul><li>mooncake.warmpath.chinacloudapi.cn</li><li>shoebox2.metrics.nsatc.net</li><li>prod3.metrics.nsatc.net</li></ul> |

* **快速路由安裝**：常見的客戶組態是定義其專屬預設路由 (0.0.0.0/0)，以強制輸出網際網路流量來替代透過內部部署方式流動。 此流量流程一定會中斷與 Azure API 管理的連線，因為已在內部部署封鎖輸出流量，或者 NAT 至無法再使用各種 Azure 端點的一組無法辨識位址。 解決方法是在子網路上定義包含 Azure API 管理的一 (或多個) 使用者定義路由 ([UDR][UDRs])。 UDR 會定義將使用的子網路特有路由，而非預設路由。
  如果可能，建議使用下列設定：
 * ExpressRoute 組態會通告 0.0.0.0/0 而且預設會使用強制通道將所有輸出流量傳送至內部部署。
 * 套用至包含 Azure API 管理的子網路之 UDR 會使用網際網路的下一個躍點類型定義 0.0.0.0/0。
 這些步驟的合併效果是子網路層級 UDR 會優先於 ExpressRoute 強制通道，因而確保來自 Azure API 管理的輸出網際網路存取。

* **透過網路虛擬設備進行路由傳送**：組態如果是使用 UDR 搭配預設路由 (0.0.0.0/0)，透過在 Azure 中執行的網路虛擬設備來路由傳送從「API 管理」子網路流向網際網路的流量，將可防止來自網際網路的管理流量流向虛擬網路子網路內所部署的「API 管理」服務執行個體。 不支援這樣的設定。

>[!WARNING]
>**未正確交叉通告從公用對等互連路徑至私人對等互連路徑之路由**的 ExpressRoute 組態不支援 Azure API 管理。 已設定公用對等互連的 ExpressRoute 組態，會收到來自 Microsoft 的一大組 Microsoft Azure IP 位址範圍的路由通告。 如果這些位址範圍在私人對等互連路徑上不正確地交叉通告，最後的結果會是來自 Azure API 管理執行個體子網路的所有輸出網路封包，都會不正確地使用強制通道傳送至客戶的內部部署網路基礎結構。 這個網路流量會中斷 Azure API 管理。 此問題的解決方案是停止從公用對等互連路徑至私人對等互連路徑的交叉通告路由。


## <a name="troubleshooting"> </a>疑難排解
* **初始安裝**：若未能成功地將 API 管理服務初始部署到子網路，建議您先將虛擬機器部署到相同的子網路。 接下來，再將桌面遠端連線到虛擬機器，並驗證您可以連線到 Azure 訂用帳戶中的下列其中一個資源
    * Azure 儲存體 Blob
    * 連接字串
    * Azure 儲存體資料表

 > [!IMPORTANT]
 > 驗證過連線能力後，請務必先移除子網路中部署的所有資源，再將 API 管理部署至子網路。

* **累加式更新**：對您的網路進行變更時，請參閱 [NetworkStatus API](https://docs.microsoft.com/rest/api/apimanagement/networkstatus)，以確認 API 管理服務未遺失相依之任何關鍵資源的存取權。 連線狀態應該每隔 15 分鐘更新一次。

* **資源導覽連結**：在部署到 Resource Manager 樣式 VNet 子網路時，API 管理會藉由建立資源導覽連結來保留子網路。 如果子網路已包含來自不同提供者的資源，部署將會**失敗**。 同樣地，當您將 API 管理服務移至不同子網路或將它刪除時，我們也會移除該資源導覽連結。

* **從 Azure 入口網站測試 API**：從 Azure 入口網站測試 API 時，您的 API 管理執行個體會與內部 VNet 整合，且在 VNet 上設定的 DNS 伺服器將用於名稱解析。 如果您從 Azure 入口網站進行測試時出現 404，請確定 VNet 的 DNS 伺服器可以正確解析您的 API 管理執行個體的主機名稱。 

## <a name="subnet-size"> </a> 子網路大小需求
Azure 會在每個子網路中保留一些 IP 位址，但這些位址無法使用。 子網路的第一個和最後一個 IP 位址會保留給相容的通訊協定，以及用於 Azure 服務的額外 3 個位址。 如需詳細資訊，請參閱 [在這些子網路內使用 IP 位址是否有任何限制？](../virtual-network/virtual-networks-faq.md#are-there-any-restrictions-on-using-ip-addresses-within-these-subnets)

除了 Azure VNET 基礎結構使用的 IP 位址之外，子網路中的每個 API 管理執行個體都會為進階 SKU 的每個單位使用兩個 IP 位址，或為開發人員 SKU 使用一個 IP 位址。 每個執行個體都會保留一個額外 IP 位址作為外部負載平衡器。 當您部署到內部 vnet 時，內部負載平衡器需要其他的 IP 位址。

假設計算大於子網路大小的下限，其中可部署的 API 管理為 /29，能提供 3 個 IP 位址。

## <a name="routing"> </a> 路由
+ 負載平衡的公用 IP 位址 (VIP) 會保留下來，以供存取所有服務端點。
+ 子網路 IP 範圍的 IP (DIP) 位址會用來存取 VNET 中的資源，而公用 IP 位址 (VIP) 會用來存取 VNET 之外的資源。
+ 您可以在 Azure 入口網站的 [概觀/基本資訊] 刀鋒視窗上找到負載平衡的公用 IP 位址。

## <a name="limitations"> </a>限制
* 包含「API 管理」執行個體的子網路無法包含任何其他 Azure 資源類型。
* 子網路和「API 管理」服務必須位於同一個訂用帳戶中。
* 包含「API 管理」執行個體的子網路無法跨訂用帳戶移動。
* 針對設定為內部虛擬網路模式的多區域 API 管理部署，使用者需負責管理跨多個區域的負載平衡，因為他們擁有路由。
* 由於平台限制，無法從另一個區域中全域對等互連 VNET 中的資源連線到內部模式中的 API 管理服務。 如需詳細資訊，請參閱[一個虛擬網路中的資源無法與對等互連虛擬網路中的 Azure 內部負載平衡器通訊](../virtual-network/virtual-network-manage-peering.md#requirements-and-constraints)。


## <a name="related-content"> </a>相關內容
* [使用 VPN 閘道將虛擬網路連線到後端](../vpn-gateway/vpn-gateway-about-vpngateways.md#s2smulti)
* [從不同的部署模型連接虛擬網路](../vpn-gateway/vpn-gateway-connect-different-deployment-models-powershell.md)
* [如何在 Azure API 管理中使用 API 偵測器來追蹤呼叫](api-management-howto-api-inspector.md)
* [虛擬網路常見問題集](../virtual-network/virtual-networks-faq.md)

[api-management-using-vnet-menu]: ./media/api-management-using-with-vnet/api-management-menu-vnet.png
[api-management-setup-vpn-select]: ./media/api-management-using-with-vnet/api-management-using-vnet-type.png
[api-management-setup-vpn-select]: ./media/api-management-using-with-vnet/api-management-using-vnet-select.png
[api-management-setup-vpn-add-api]: ./media/api-management-using-with-vnet/api-management-using-vnet-add-api.png
[api-management-vnet-private]: ./media/api-management-using-with-vnet/api-management-vnet-internal.png
[api-management-vnet-public]: ./media/api-management-using-with-vnet/api-management-vnet-external.png

[Enable VPN connections]: #enable-vpn
[Connect to a web service behind VPN]: #connect-vpn
[Related content]: #related-content

[UDRs]: ../virtual-network/virtual-networks-udr-overview.md
[Network Security Group]: ../virtual-network/security-overview.md
