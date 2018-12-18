---
title: 將虛擬網路連結到 ExpressRoute 線路：PowerShell：傳統：Azure | Microsoft Docs
description: 本文提供以下內容的概觀：如何使用傳統部署模型和 PowerShell 將虛擬網路 (VNet) 連結到 ExpressRoute 線路。
services: expressroute
documentationcenter: na
author: cherylmc
ms.service: expressroute
ms.topic: conceptual
ms.date: 07/27/2018
ms.author: cherylmc
ms.openlocfilehash: 10b623947b6e776c4f8f41e8424262d7f2a3e933
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39343370"
---
# <a name="connect-a-virtual-network-to-an-expressroute-circuit-using-powershell-classic"></a>使用 PowerShell 將虛擬網路連接到 ExpressRoute 線路 (傳統)
> [!div class="op_single_selector"]
> * [Azure 入口網站](expressroute-howto-linkvnet-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-linkvnet-arm.md)
> * [Azure CLI](howto-linkvnet-cli.md)
> * [影片 - Azure 入口網站](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-connection-between-your-vpn-gateway-and-expressroute-circuit)
> * [PowerShell (傳統)](expressroute-howto-linkvnet-classic.md)
>

本文將協助您使用 PowerShell 將虛擬網路 (VNet) 連結到 Azure ExpressRoute 線路。 單一 VNet 最多可連結到四個 ExpressRoute 線路。 使用本文中的步驟來建立您要連線之每個 ExpressRoute 線路的新連結。 ExpressRoute 線路可以位於相同的訂用帳戶、不同的訂用帳戶或兩者的混合。 本文適用於使用傳統部署模型所建立的虛擬網路。

您最多可以將 10 個虛擬網路連結至 ExpressRoute 電路。 所有的虛擬網路都必須位於同一個地理區域。 如果您啟用 ExpressRoute 進階版附加元件，則可將更大量的虛擬網路連結到您的 ExpressRoute 線路，或是連結位於其他地理區域的虛擬網路。 如需進階版附加元件的詳細資訊，請參閱[常見問題集](expressroute-faqs.md)。

[!INCLUDE [expressroute-classic-end-include](../../includes/expressroute-classic-end-include.md)]

**關於 Azure 部署模型**

[!INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

## <a name="configuration-prerequisites"></a>組態必要條件

* 開始設定之前，請先檢閱[必要條件](expressroute-prerequisites.md)、[路由需求](expressroute-routing.md)及[工作流程](expressroute-workflows.md)。
* 您必須擁有作用中的 ExpressRoute 線路。
   * 遵循指示來 [建立 ExpressRoute 線路](expressroute-howto-circuit-classic.md) ，並由您的連線提供者來啟用該線路。
   * 確定您已針對循環設定了 Azure 私用對等。 請參閱 [設定路由](expressroute-howto-routing-classic.md) 一文，以取得路由指示。
   * 請確定已設定 Azure 私用對等，且已開啟您的網路與 Microsoft 之間的 BGP 對等，讓您可以啟用端對端連線。
   * 您必須有已建立且完整佈建的虛擬網路和虛擬網路閘道。 請遵循指示 [設定 ExpressRoute 的虛擬網路](expressroute-howto-vnet-portal-classic.md)。

### <a name="download-the-latest-powershell-cmdlets"></a>下載最新的 PowerShell Cmdlet

安裝最新版的 Azure 服務管理 (SM) PowerShell 模組和 ExpressRoute 模組。 當您使用下列範例時，請注意版本號碼 (此範例中為 5.1.1) 會因為發行較新的 Cmdlet 版本而變更。

```powershell
Import-Module 'C:\Program Files\WindowsPowerShell\Modules\Azure\5.1.1\Azure\Azure.psd1'
Import-Module 'C:\Program Files\WindowsPowerShell\Modules\Azure\5.1.1\ExpressRoute\ExpressRoute.psd1'
```

如需 Azure PowerShell 的詳細資訊，請參閱[開始使用 Azure PowerShell Cmdlet](/powershell/azure/overview)，來取得如何設定您的電腦以使用 Azure PowerShell 模組的逐步指導方針。

### <a name="sign-in"></a>登入

若要登入您的 Azure 帳戶，請使用下列範例：

1. 以提高的權限開啟 PowerShell 主控台並連接到您的帳戶。

  ```powershell
  Connect-AzureRmAccount
  ```
2. 檢查帳戶的訂用帳戶。

  ```powershell
  Get-AzureRmSubscription
  ```
3. 如果您有多個訂用帳戶，請選取您要使用的訂用帳戶。

  ```powershell
  Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"
  ```

4. 接下來，使用下列 Cmdlet，將您的 Azure 訂用帳戶新增到 PowerShell，以供傳統部署模型使用。

  ```powershell
  Add-AzureAccount
  ```

## <a name="connect-a-virtual-network-in-the-same-subscription-to-a-circuit"></a>將相同訂用帳戶中的虛擬網路連接到線路
您可以使用下列 Cmdlet，將虛擬網路連結到 ExpressRoute 線路。 執行 Cmdlet 之前，請確定您已建立虛擬網路閘道，並準備好進行連結。

```powershell
New-AzureDedicatedCircuitLink -ServiceKey "*****************************" -VNetName "MyVNet"
Provisioned
```
    
## <a name="remove-a-virtual-network-link-to-a-circuit"></a>移除線路的虛擬網路連結
您可以使用下列 Cmdlet 移除 ExpressRoute 線路的虛擬網路連結。 請確定已針對特定的虛擬網路選取目前的訂用帳戶。 

```powershell
Remove-AzureDedicatedCircuitLink -ServiceKey "*****************************" -VNetName "MyVNet"
```
 

## <a name="connect-a-virtual-network-in-a-different-subscription-to-a-circuit"></a>將不同訂用帳戶中的虛擬網路連接到線路
您可以讓多個訂用帳戶共用 ExpressRoute 線路。 下圖顯示簡單的圖解，示範多個訂用帳戶共用 ExpressRoute 線路的方式。

大型雲端內的每個較小型雲端，會用來代表屬於組織內不同部門的訂用帳戶。 組織內的每個部門都可以使用自己的訂用帳戶來部署它們的服務，但可共用單一 ExpressRoute 線路，以連接回內部部署網路。 單一部門 (在此範例中：IT) 可以擁有 ExpressRoute 循環。 組織內的其他訂用帳戶可以使用 ExpressRoute 電路。

> [!NOTE]
> ExpressRoute 循環擁有者需支付專用循環的連線和頻寬費用。 所有虛擬網路都會共用相同的頻寬。
> 
> 

![跨訂用帳戶的連線能力](./media/expressroute-howto-linkvnet-classic/cross-subscription.png)

### <a name="administration"></a>系統管理
「線路擁有者」  是訂用帳戶的管理員/共同管理員，而 ExpressRoute 線路即建立於該訂用帳戶中。 線路擁有者可以授權其他訂用帳戶的管理員/共同管理員 (工作流程圖中稱為「線路使用者」 ) 來使用他們擁有的專用線路。 獲得使用組織 ExpressRoute 線路的授權後，這些線路使用者就可以將其訂用帳戶中的虛擬網路連結到 ExpressRoute 線路。

電路擁有者能夠隨時修改及撤銷授權。 如果撤銷授權，則在存取權遭撤銷的訂用帳戶中，所有連結均會被刪除。

### <a name="circuit-owner-operations"></a>循環擁有者作業

**建立授權**

電路擁有者可授權其他訂用帳戶管理員使用指定的電路。 在下列範例中，線路管理員 (Contoso IT) 讓另一個訂用帳戶的管理員 (Dev-Test) 可將最多 2 個虛擬網路連結到線路。 Contoso IT 的系統管理員的做法是指定 Dev-Test Microsoft 識別碼。 此 Cmdlet 不會傳送電子郵件給指定的 Microsoft ID。 電路擁有者必須明確通知其他訂用帳戶擁有者，告知授權已完成。

```powershell
New-AzureDedicatedCircuitLinkAuthorization -ServiceKey "**************************" -Description "Dev-Test Links" -Limit 2 -MicrosoftIds 'devtest@contoso.com'
```

  傳回：

  ```powershell
  Description         : Dev-Test Links
  Limit               : 2
  LinkAuthorizationId : **********************************
  MicrosoftIds        : devtest@contoso.com
  Used                : 0
  ```

**檢閱授權**

線路擁有者可以藉由執行下列 Cmdlet，來檢閱特定線路上發出的所有授權：

```powershell
Get-AzureDedicatedCircuitLinkAuthorization -ServiceKey: "**************************"
```
  傳回：

  ```powershell
  Description         : EngineeringTeam
  Limit               : 3
  LinkAuthorizationId : ####################################
  MicrosoftIds        : engadmin@contoso.com
  Used                : 1

  Description         : MarketingTeam
  Limit               : 1
  LinkAuthorizationId : @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
  MicrosoftIds        : marketingadmin@contoso.com
  Used                : 0

  Description         : Dev-Test Links
  Limit               : 2
  LinkAuthorizationId : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&
  MicrosoftIds        : salesadmin@contoso.com
  Used                : 2
  ```

**更新授權**

線路擁有者可以使用下列 Cmdlet 來修改授權：

```powershell
Set-AzureDedicatedCircuitLinkAuthorization -ServiceKey "**************************" -AuthorizationId "&&&&&&&&&&&&&&&&&&&&&&&&&&&&"-Limit 5
```

  傳回：

  ```powershell
  Description         : Dev-Test Links
  Limit               : 5
  LinkAuthorizationId : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&
  MicrosoftIds        : devtest@contoso.com
  Used                : 0
  ```

**刪除授權**

線路擁有者可以使用下列 Cmdlet 來撤銷/刪除使用者的授權：

```powershell
Remove-AzureDedicatedCircuitLinkAuthorization -ServiceKey "*****************************" -AuthorizationId "###############################"
```

### <a name="circuit-user-operations"></a>循環使用者作業

**檢閱授權**

線路使用者可以使用下列 Cmdlet 來檢閱授權：

```powershell
Get-AzureAuthorizedDedicatedCircuit
```

  傳回：

  ```powershell
  Bandwidth                        : 200
  CircuitName                      : ContosoIT
  Location                         : Washington DC
  MaximumAllowedLinks              : 2
  ServiceKey                       : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&
  ServiceProviderName              : equinix
  ServiceProviderProvisioningState : Provisioned
  Status                           : Enabled
  UsedLinks                        : 0
  ```

**兌換連結授權**

線路使用者可以執行下列 Cmdlet 來兌換連結授權：

```powershell
New-AzureDedicatedCircuitLink –servicekey "&&&&&&&&&&&&&&&&&&&&&&&&&&" –VnetName 'SalesVNET1'
```

  傳回：

  ```powershell
  State VnetName
  ----- --------
  Provisioned SalesVNET1
  ```

在虛擬網路的新連結訂用帳戶中執行此命令：

```powershell
New-AzureDedicatedCircuitLink -ServiceKey "*****************************" -VNetName "MyVNet"
```

## <a name="next-steps"></a>後續步驟

如需有關 ExpressRoute 的詳細資訊，請參閱 [ExpressRoute 常見問題集](expressroute-faqs.md)。