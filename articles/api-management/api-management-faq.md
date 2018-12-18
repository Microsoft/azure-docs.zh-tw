---
title: Azure API 管理常見問題集 | Microsoft Docs
description: 了解 Azure API 管理中常見問題集 (FAQ)、模式和最佳做法的答案。
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 2fa193cd-ea71-4b33-a5ca-1f55e5351e23
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/19/2017
ms.author: apimpm
ms.openlocfilehash: 7082edf43e53766d32bd967d0b894f6439eaa492
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "42144803"
---
# <a name="azure-api-management-faqs"></a>Azure API 管理常見問題集
得到 Azure API 管理常見問題、模式和最佳作法的答案。

## <a name="contact-us"></a>與我們連絡
* [如何向 Microsoft Azure API 管理小組詢問問題？](#how-can-i-ask-the-microsoft-azure-api-management-team-a-question)

## <a name="frequently-asked-questions"></a>常見問題集
* [功能預覽中是什麼意思？](#what-does-it-mean-when-a-feature-is-in-preview)
* [如何保護 API 管理閘道與後端服務之間的連線安全？](#how-can-i-secure-the-connection-between-the-api-management-gateway-and-my-back-end-services)
* [如何將我的 API 管理服務執行個體複製到新的執行個體？](#how-do-i-copy-my-api-management-service-instance-to-a-new-instance)
* [我可以透過程式設計方式管理我的 API 管理執行個體嗎？](#can-i-manage-my-api-management-instance-programmatically)
* [如何將使用者新增至 Administrators 群組？](#how-do-i-add-a-user-to-the-administrators-group)
* [為什麼「我想要新增」的原則無法用於原則編輯器？](#why-is-the-policy-that-i-want-to-add-unavailable-in-the-policy-editor)
* [如何在單一 API 中設定多個環境？](#how-do-i-set-up-multiple-environments-in-a-single-api)
* [可以使用 SOAP 搭配 API 管理嗎？](#can-i-use-soap-with-api-management)
* [API 管理閘道 IP 位址是否固定？我可以用在防火牆規則嗎？](#is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules)
* [可以設定具有 AD FS 安全性的 OAUth 2.0 授權伺服器嗎？](#can-i-configure-an-oauth-20-authorization-server-with-adfs-security)
* [API 管理在部署到多個地理位置時使用何種路由方法？](#what-routing-method-does-api-management-use-in-deployments-to-multiple-geographic-locations)
* [可以使用 Azure Resource Manager 範本建立 API 管理服務執行個體嗎？](#can-i-use-an-azure-resource-manager-template-to-create-an-api-management-service-instance)
* [是否可以對後端使用自我簽署的 SSL 憑證？](#can-i-use-a-self-signed-ssl-certificate-for-a-back-end)
* [為什麼我在嘗試複製 GIT 儲存機制時會發生驗證失敗？](#why-do-i-get-an-authentication-failure-when-i-try-to-clone-a-git-repository)
* [API 管理是否能搭配 Azure ExpressRoute 運作？](#does-api-management-work-with-azure-expressroute)
* [為什麼將「API 管理」部署到 Resource Manager 樣式的 VNET 中時，我們需要在這些 VNET 中有一個專用子網路？](#why-do-we-require-a-dedicated-subnet-in-resource-manager-style-vnets-when-api-management-is-deployed-into-them)
* [將「API 管理」部署到 VNET 中時，所需的子網路大小下限是多少？](#what-is-the-minimum-subnet-size-needed-when-deploying-api-management-into-a-vnet)
* [可以將 API 管理服務從某一個訂用帳戶移至另一個嗎？](#can-i-move-an-api-management-service-from-one-subscription-to-another)
* [匯入 API 有任何限制或已知的問題嗎？](#are-there-restrictions-on-or-known-issues-with-importing-my-api)

### <a name="how-can-i-ask-the-microsoft-azure-api-management-team-a-question"></a>如何向 Microsoft Azure API 管理小組詢問問題？
您可以使用下列其中一個選項與我們連絡︰

* 將問題張貼在我們的 [API 管理 MSDN 論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=azureapimgmt)。
* 將電子郵件傳送至 <mailto:apimgmt@microsoft.com>。
* 在 [Azure 意見反應論壇](https://feedback.azure.com/forums/248703-api-management)將功能要求傳送給我們。

### <a name="what-does-it-mean-when-a-feature-is-in-preview"></a>功能預覽中是什麼意思？
當功能預覽中時，這表示我們正積極尋求您對於功能運作情況的意見反應。 預覽中的功能具備完整功能，但我們可能會進行中斷變更，以回應客戶的意見反應。 我們建議您不要依賴您生產環境中處於預覽狀態的功能。 如果您對於預覽功能有任何意見反應，請透過[如何向 Microsoft Azure API 管理小組詢問問題？](#how-can-i-ask-the-microsoft-azure-api-management-team-a-question)的其中一個連絡選項，讓我們知道。

### <a name="how-can-i-secure-the-connection-between-the-api-management-gateway-and-my-back-end-services"></a>如何保護 API 管理閘道與後端服務之間的連線安全？
您有幾個選項可以保護 API 管理閘道與後端服務之間的連線安全。 您可以：

* 使用 HTTP 基本驗證。 如需詳細資訊，請參閱[匯入和發佈您的第一個 API](import-and-publish.md)。
* 依照[如何使用 Azure API 管理中的用戶端憑證驗證保護後端服務](api-management-howto-mutual-certificates.md)所述，使用 SSL 相互驗證。
* 在您的後端服務上使用 IP 允許清單。 在 API 管理的所有層級中，閘道的 IP 位址會維持不變，但有一些[注意事項](#is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules)。 您可以設定您的允許清單來允許此 IP 位址。 您可以在 Azure 入口網站中的 [儀表板] 上取得API 管理執行個體的 IP 位址。
* 將您的 API 管理執行個體連接至 Azure 虛擬網路。

### <a name="how-do-i-copy-my-api-management-service-instance-to-a-new-instance"></a>如何將我的 API 管理服務執行個體複製到新的執行個體？
如果您要將 API 管理執行個體複製到新的執行個體，您有幾個選項可用。 您可以：

* 使用 API 管理的備份與還原功能。 如需詳細資訊，請參閱[如何在 Azure API 管理中使用服務備份和還原實作災害復原](api-management-howto-disaster-recovery-backup-restore.md)。
* 使用 [API 管理 REST API](https://msdn.microsoft.com/library/azure/dn776326.aspx) 建立自己的備份和還原功能。 使用 REST API，從您想要的服務執行個體儲存和還原實體。
* 使用 Git 下載服務組態，然後將它上傳至新的執行個體。 如需詳細資訊，請參閱[如何使用 Git 儲存和設定您的 API 管理服務組態](api-management-configuration-repository-git.md)。

### <a name="can-i-manage-my-api-management-instance-programmatically"></a>我可以透過程式設計方式管理我的 API 管理執行個體嗎？
可以，您可以使用下列各項，以程式設計方式管理 API 管理：

* [API 管理 REST API](https://msdn.microsoft.com/library/azure/dn776326.aspx)。
* [Microsoft Azure ApiManagement 服務管理資源庫 SDK](http://aka.ms/apimsdk)。
* [服務部署](https://docs.microsoft.com/powershell/module/wds)和[服務管理](https://docs.microsoft.com/powershell/azure/servicemanagement/overview) PowerShell Cmdlet。

### <a name="how-do-i-add-a-user-to-the-administrators-group"></a>如何將使用者新增至 Administrators 群組？
將使用者新增至 Administrators 群組的步驟如下：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 移至包含您要更新之 API 管理執行個體的資源群組。
3. 在 API 管理中，將 [Api 管理參與者] 角色指派給使用者。

新增的參與者現在即可使用 Azure PowerShell [Cmdlet](https://docs.microsoft.com/powershell/azure/overview)。 以系統管理員身分登入的步驟如下︰

1. 使用 `Connect-AzureRmAccount` Cmdlet 進行登入。
2. 使用 `Set-AzureRmContext -SubscriptionID <subscriptionGUID>` 將內容設定為具有服務的訂用帳戶。
3. 使用 `Get-AzureRmApiManagementSsoToken -ResourceGroupName <rgName> -Name <serviceName>` 取得單一登入 URL。
4. 使用此 URL 來存取系統管理入口網站。

### <a name="why-is-the-policy-that-i-want-to-add-unavailable-in-the-policy-editor"></a>為什麼「我想要新增」的原則無法用於原則編輯器？
如果您想要新增的原則在原則編輯器中呈暗灰色，請確定您是在該原則的正確範圍內。 每個原則陳述式都是為您在特定範圍和原則區段中使用所設計。 若要檢閱原則的原則區段和範圍，請參閱 [API 管理原則](https://msdn.microsoft.com/library/azure/dn894080.aspx)中原則的「使用方式」一節。

### <a name="how-do-i-set-up-multiple-environments-in-a-single-api"></a>如何在單一 API 中設定多個環境？
若要在單一 API 中設定多個環境 (例如一個測試環境和一個生產環境)，您有兩個選項。 您可以：

* 在相同租用戶上裝載不同的 API。
* 在不同租用戶上裝載相同的 API。

### <a name="can-i-use-soap-with-api-management"></a>可以使用 SOAP 搭配 API 管理嗎？
現在可以使用 [SOAP 傳遞](http://blogs.msdn.microsoft.com/apimanagement/2016/10/13/soap-pass-through/)支援。 系統管理員可以匯入其 SOAP 服務的 WSDL，而 Azure API 管理將會建立 SOAP 前端。 開發人員入口網站文件、測試主控台、原則和分析全都適用於 SOAP 服務。

### <a name="is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules"></a>API 管理閘道 IP 位址是否固定？ 我可以用在防火牆規則嗎？
在 API 管理的所有層級中，API 管理租用戶的公用 IP 位址 (VIP) 在租用戶的存留期內是靜態的，但有一些例外狀況。 在下列情況下，IP 位址會變更︰

* 服務遭到刪除，然後又重新建立。
* 服務訂用帳戶遭到[暫止](https://github.com/Azure/azure-resource-manager-rpc/blob/master/v1.0/subscription-lifecycle-api-reference.md#subscription-states)或[警告](https://github.com/Azure/azure-resource-manager-rpc/blob/master/v1.0/subscription-lifecycle-api-reference.md#subscription-states) (例如，因為未付款)，然後又恢復。
* 您新增或移除「Azure 虛擬網路」(您只能在「開發人員」或「進階」層使用「虛擬網路」)。

對於多重區域的部署，如果區域被清空而後恢復，區域位址則會變更 (您只能在進階層使用多重區域部署)。

針對多重區域部署設定的進階層租用戶，每個區域會被指派一個公用 IP 位址。

您可以在 Azure 入口網站的租用戶頁面上取得您的 IP 位址 (或在多重區域部署中取得多個位址)。

### <a name="can-i-configure-an-oauth-20-authorization-server-with-ad-fs-security"></a>可以設定具有 AD FS 安全性的 OAUth 2.0 授權伺服器嗎？
若要了解如何使用 Active Directory Federation Services (AD FS) 安全性來設定 OAuth 2.0 授權伺服器，請參閱[在 API 管理中使用 ADFS](https://phvbaars.wordpress.com/2016/02/06/using-adfs-in-api-management/)。

### <a name="what-routing-method-does-api-management-use-in-deployments-to-multiple-geographic-locations"></a>API 管理在部署到多個地理位置時使用何種路由方法？
API 管理會在部署到多個地理位置時，使用[效能流量路由方法](../traffic-manager/traffic-manager-routing-methods.md#performance)。 連入流量會傳送至最接近的 API 閘道。 如果一個區域離線，則連入流量會自動路由傳送至下一個最接近的閘道。 深入了解[流量管理員路由方法](../traffic-manager/traffic-manager-routing-methods.md)中的路由方法。

### <a name="can-i-use-an-azure-resource-manager-template-to-create-an-api-management-service-instance"></a>可以使用 Azure Resource Manager 範本建立 API 管理服務執行個體嗎？
是。 請參閱 [Azure API 管理服務](http://aka.ms/apimtemplate)快速入門範本。

### <a name="can-i-use-a-self-signed-ssl-certificate-for-a-back-end"></a>是否可以對後端使用自我簽署的 SSL 憑證？
是。 這可以透過 PowerShell 或直接提交至 API 來完成。 這將會停用信任鏈結驗證，在從 API 管理對後端服務進行通訊時，還可讓您使用自我簽署或私人簽署的憑證。

#### <a name="powershell-method"></a>Powershell 方法 ####
使用 [`New-AzureRmApiManagementBackend` ](https://docs.microsoft.com/powershell/module/azurerm.apimanagement/new-azurermapimanagementbackend) (適用於新的後端) 或 [ `Set-AzureRmApiManagementBackend` ](https://docs.microsoft.com/powershell/module/azurerm.apimanagement/set-azurermapimanagementbackend) (適用於現有的後端) PowerShell Cmdlet，並將 `-SkipCertificateChainValidation` 參數設定為 `True`。 

```
$context = New-AzureRmApiManagementContext -resourcegroup 'ContosoResourceGroup' -servicename 'ContosoAPIMService'
New-AzureRmApiManagementBackend -Context  $context -Url 'https://contoso.com/myapi' -Protocol http -SkipCertificateChainValidation $true
```

#### <a name="direct-api-update-method"></a>直接 API 更新方法 ####
1. 使用 API 管理建立[後端](https://msdn.microsoft.com/library/azure/dn935030.aspx)實體。       
2. 將 **skipCertificateChainValidation** 屬性設定為 **true**。     
3. 如果您不想再允許自我簽署的憑證，請刪除後端實體或將 **skipCertificateChainValidation** 屬性設定為 **false**。

### <a name="why-do-i-get-an-authentication-failure-when-i-try-to-clone-a-git-repository"></a>為什麼我在嘗試複製 Git 儲存機制時會發生驗證失敗？
如果您使用 Git 認證管理員，或如果您嘗試使用 Visual Studio 複製 Git 儲存機制，您可能會遇到 Windows 認證對話方塊的已知問題。 此對話方塊會將密碼長度限制為 127 個字元，並且截斷 Microsoft 所產生的密碼。 我們正在努力縮短密碼。 目前，請使用 Git Bash 來複製 Git 儲存機制。

### <a name="does-api-management-work-with-azure-expressroute"></a>API 管理是否能搭配 Azure ExpressRoute 運作？
是。 API 管理能搭配 Azure ExpressRoute 運作。

### <a name="why-do-we-require-a-dedicated-subnet-in-resource-manager-style-vnets-when-api-management-is-deployed-into-them"></a>為什麼將「API 管理」部署到 Resource Manager 樣式的 VNET 中時，我們需要在這些 VNET 中有一個專用子網路？
「API 管理」的專用子網路需求來自於一個事實，就是它是建置在「傳統」(PAAS V1 層) 部署模型的基礎上。 雖然我們可以部署到 Resource Manager VNET (V2 層) 中，但是會有後果。 Azure 中的「傳統」部署模型並未與 Resource Manager 模型緊密結合，因此，如果您在 V2 層中建立一項資源，V1 層並不會知道它，而問題就可能發生，例如「API 管理」嘗試使用已經配置給某個 NIC (建置在 V2 上) 的 IP。
若要深入了解 Azure 中「傳統」模型與 Resource Manager 模型的差異，請參閱[部署模型中的差異](../azure-resource-manager/resource-manager-deployment-model.md)。

### <a name="what-is-the-minimum-subnet-size-needed-when-deploying-api-management-into-a-vnet"></a>將「API 管理」部署到 VNET 中時，所需的子網路大小下限是多少？
部署「API 管理」所需的子網路大小下限是 [/29](../virtual-network/virtual-networks-faq.md#configuration)，這是 Azure 支援的子網路大小下限。

### <a name="can-i-move-an-api-management-service-from-one-subscription-to-another"></a>可以將 API 管理服務從某一個訂用帳戶移至另一個嗎？
是。 若要了解作法，請參閱[將資源移至新的資源群組或訂用帳戶](../azure-resource-manager/resource-group-move-resources.md)。

### <a name="are-there-restrictions-on-or-known-issues-with-importing-my-api"></a>匯入 API 有任何限制或已知的問題嗎？
Open API(Swagger)、WSDL 及 WADL 格式的[已知問題和限制](api-management-api-import-restrictions.md)。
