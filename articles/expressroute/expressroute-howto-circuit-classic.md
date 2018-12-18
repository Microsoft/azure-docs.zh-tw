---
title: 修改 ExpressRoute 線路︰PowerShell：Azure 傳統| Microsoft Docs
description: 本文將逐步引導您檢查狀態、更新或刪除，以及取消佈建 ExpressRoute 傳統部署模型線路。
services: expressroute
author: ganesr
ms.service: expressroute
ms.topic: conceptual
ms.date: 07/26/2018
ms.author: ganesr;cherylmc
ms.openlocfilehash: 407782ff59147f227f5f34bc3318333093b4f57e
ms.sourcegitcommit: 068fc623c1bb7fb767919c4882280cad8bc33e3a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39283566"
---
# <a name="modify-an-expressroute-circuit-using-powershell-classic"></a>使用 PowerShell 修改 ExpressRoute 線路 (傳統)

> [!div class="op_single_selector"]
> * [Azure 入口網站](expressroute-howto-circuit-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-circuit-arm.md)
> * [Azure CLI](howto-circuit-cli.md)
> * [影片 - Azure 入口網站](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit)
> * [PowerShell (傳統)](expressroute-howto-circuit-classic.md)
>

本文也會示範如何檢查狀態、更新或刪除和取消佈建 ExpressRoute 線路。

[!INCLUDE [expressroute-classic-end-include](../../includes/expressroute-classic-end-include.md)]

**關於 Azure 部署模型**

[!INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

## <a name="before-you-begin"></a>開始之前

安裝最新版的 Azure 服務管理 (SM) PowerShell 模組和 ExpressRoute 模組。  當您使用下列範例時，請注意版本號碼 (此範例中為 5.1.1) 會因為發行較新的版本而變更。

```powershell
Import-Module 'C:\Program Files\WindowsPowerShell\Modules\Azure\5.1.1\Azure\Azure.psd1'
Import-Module 'C:\Program Files\WindowsPowerShell\Modules\Azure\5.1.1\ExpressRoute\ExpressRoute.psd1'
```

如需 Azure PowerShell 的詳細資訊，請參閱[開始使用 Azure PowerShell Cmdlet](/powershell/azure/overview)，來取得如何設定您的電腦以使用 Azure PowerShell 模組的逐步指引。

若要登入您的 Azure 帳戶，請使用下列範例：

1. 以提高的權限開啟 PowerShell 主控台並連接到您的帳戶。 使用下列範例來協助您連接：

  ```powershel
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

## <a name="get-the-status-of-a-circuit"></a>取得線路狀態

您隨時可以使用 `Get-AzureCircuit` Cmdlet 擷取此資訊。 執行呼叫時，若未指定任何參數，將會列出所有線路。

```powershell
Get-AzureDedicatedCircuit

Bandwidth                        : 200
CircuitName                      : MyTestCircuit
Location                         : Silicon Valley
ServiceKey                       : *********************************
ServiceProviderName              : equinix
ServiceProviderProvisioningState : Provisioned
Sku                              : Standard
Status                           : Enabled

Bandwidth                        : 1000
CircuitName                      : MyAsiaCircuit
Location                         : Singapore
ServiceKey                       : #################################
ServiceProviderName              : equinix
ServiceProviderProvisioningState : Provisioned
Sku                              : Standard
Status                           : Enabled
```

您可以將服務金鑰當作參數傳遞給呼叫，以取得特定 ExpressRoute 線路的詳細資訊。

```powershell
Get-AzureDedicatedCircuit -ServiceKey "*********************************"

Bandwidth                        : 200
CircuitName                      : MyTestCircuit
Location                         : Silicon Valley
ServiceKey                       : *********************************
ServiceProviderName              : equinix
ServiceProviderProvisioningState : Provisioned
Sku                              : Standard
Status                           : Enabled
```

您可以執行下列範例來取得所有參數的詳細描述：

```powershell
get-help get-azurededicatedcircuit -detailed
```

## <a name="modify-a-circuit"></a>修改線路

您可以修改 ExpressRoute 線路的某些屬性，而不會影響連線。

您可以執行下列工作，而無需中途停機：

* 啟用或停用 ExpressRoute 線路的 ExpressRoute 進階附加元件。
* 只要連接埠有可用的容量，就增加 ExpressRoute 線路的頻寬。 不支援將線路的頻寬降級。 
* 將計量方案從 [計量付費] 變更為 [無限制]。 不支援將計量方案從 [無限制資料] 變更為 [已計量資料]。
* 您可以啟用和停用 [允許傳統作業] 。

如需限制的詳細資訊，請參閱 [ExpressRoute 常見問題集](expressroute-faqs.md) 。

### <a name="enable-the-expressroute-premium-add-on"></a>啟用 ExpressRoute 進階附加元件

您可以使用下列 PowerShell Cmdlet，為現有的線路啟用 ExpressRoute Premium 附加元件：

```powershell
Set-AzureDedicatedCircuitProperties -ServiceKey "*********************************" -Sku Premium

Bandwidth                        : 1000
CircuitName                      : TestCircuit
Location                         : Silicon Valley
ServiceKey                       : *********************************
ServiceProviderName              : equinix
ServiceProviderProvisioningState : Provisioned
Sku                              : Premium
Status                           : Enabled
```

您的線路現在將啟用 ExpressRoute Premium 附加功能。 一旦順利執行命令，就會立即開始針對進階附加元件功能計費。

### <a name="disable-the-expressroute-premium-add-on"></a>停用 ExpressRoute 進階附加元件

> [!IMPORTANT]
> 如果您使用的資源超出標準線路所允許的數量，這項作業可能會失敗。
> 
> 

#### <a name="considerations"></a>考量

* 從高階降級為標準之前，請確定連結至線路的虛擬網路數目小於 10。 如果您不這樣做，更新要求就會失敗，且會被以進階費率計費。
* 您必須取消連結其他地理政治區域中的所有虛擬網路。 如果您不這樣做，更新要求就會失敗，且會被以進階費率計費。
* 就私用對等設定而言，路由表必須少於 4000 個路由。 如果路由表大小超過 4000 個路由，BGP 工作階段將會中斷，而且在通告的首碼數目降到 4000 以下之前不會重新啟用。

#### <a name="to-disable-the-premium-add-on"></a>停用進階附加元件

您可以使用下列 PowerShell Cmdlet，為現有的線路停用 ExpressRoute Premium 附加元件：

```powershell

Set-AzureDedicatedCircuitProperties -ServiceKey "*********************************" -Sku Standard

Bandwidth                        : 1000
CircuitName                      : TestCircuit
Location                         : Silicon Valley
ServiceKey                       : *********************************
ServiceProviderName              : equinix
ServiceProviderProvisioningState : Provisioned
Sku                              : Standard
Status                           : Enabled
```

### <a name="update-the-expressroute-circuit-bandwidth"></a>更新 ExpressRoute 線路頻寬

請查閱 [ExpressRoute 常見問題集](expressroute-faqs.md) ，以取得提供者支援的頻寬選項。 只要實體連接埠 (您的線路在此建立) 允許，您可以選擇大於現有線路的任何大小。

> [!IMPORTANT]
> 如果現有的連接埠上沒有足夠的容量，您可能必須重新建立 ExpressRoute 線路。 如果該位置已無額外的容量，您無法升級線路。
>
> 降低 ExpressRoute 線路的頻寬時必須中斷運作。 頻寬降級需要取消佈建 ExpressRoute 線路，然後重新佈建新的 ExpressRoute 線路。
> 
> 

#### <a name="resize-a-circuit"></a>調整電路大小

一旦決定需要的大小後，您可以使用下列命令來調整線路大小。

```powershell
Set-AzureDedicatedCircuitProperties -ServiceKey ********************************* -Bandwidth 1000

Bandwidth                        : 1000
CircuitName                      : TestCircuit
Location                         : Silicon Valley
ServiceKey                       : *********************************
ServiceProviderName              : equinix
ServiceProviderProvisioningState : Provisioned
Sku                              : Standard
Status                           : Enabled
```

一旦在 Microsoft 端增加您的線路大小，您必須連絡連線提供者，將他們的設定更新為符合這項變更。 從這個階段起，會開始以更新後的頻寬選項計算費用。

如果增加線路頻寬時出現下列錯誤，則表示建立現有線路的實體連接埠所剩的頻寬不足。 您必須刪除此線路，並建立所需大小的新線路。

```powershell
Set-AzureDedicatedCircuitProperties : InvalidOperation : Insufficient bandwidth available to perform this circuit
update operation
At line:1 char:1
+ Set-AzureDedicatedCircuitProperties -ServiceKey ********************* ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  + CategoryInfo          : CloseError: (:) [Set-AzureDedicatedCircuitProperties], CloudException
  + FullyQualifiedErrorId : Microsoft.WindowsAzure.Commands.ExpressRoute.SetAzureDedicatedCircuitPropertiesCommand
```

## <a name="deprovision-and-delete-a-circuit"></a>取消佈建及刪除線路

### <a name="considerations"></a>考量

* 您必須取消連結 ExpressRoute 線路的所有虛擬網路，此作業才會成功。 如果此作業失敗，請檢查您是否有任何虛擬網路連結至線路。
* 如果 ExpressRoute 線路服務提供者佈建狀態為 **Provisioning** 或 **Provisioned**，您就必須與服務提供者一起合作，取消佈建他們那邊的線路。 我們會繼續保留資源並向您收取費用，直到線路服務提供者完成取消佈建並通知我們。
* 若服務提供者已取消佈建線路 (服務提供者佈建狀態設定為 [未佈建])，您便可以刪除線路。 這樣會停止針對線路計費。

#### <a name="delete-a-circuit"></a>刪除電路

您可以執行下列命令來刪除 ExpressRoute 線路：

```powershell
Remove-AzureDedicatedCircuit -ServiceKey "*********************************"
```