---
title: 管理 Azure 保留的 IP 位址 (傳統) - PowerShell | Microsoft Docs
description: 了解保留的 IP 位址 (傳統)，以及如何使用 PowerShell 管理這些 IP 位址。
services: virtual-network
documentationcenter: na
author: genlin
manager: cshepard
editor: tysonn
ms.assetid: 34652a55-3ab8-4c2d-8fb2-43684033b191
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/10/2016
ms.author: genli
ms.openlocfilehash: 25fe3c5361ff58f8d62d5d083b7a69f517d2a267
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31798949"
---
# <a name="reserved-ip-addresses-classic"></a>保留的 IP 位址 (傳統)

> [!div class="op_single_selector"]
> * [Azure 入口網站](virtual-network-deploy-static-pip-arm-portal.md)
> * [PowerShell](virtual-network-deploy-static-pip-arm-ps.md)
> * [Azure CLI](virtual-network-deploy-static-pip-arm-cli.md)
> * [PowerShell (傳統)](virtual-networks-reserved-public-ip.md)

Azure 中的 IP 位址分為兩個類別：動態和保留。 依預設由 Azure 管理的公用 IP 位址是動態的。 這表示當資源關閉或停止 (解除配置) 時，用於所指定雲端服務的 IP 位址 (VIP) 或用來直接存取 VM 或角色執行個體的 IP 位址 (ILPIP) 可以隨時變更。

若要防止 IP 位址變更，您可以保留 IP 位址。 保留的 IP 只能用來作為 VIP，用以確保在即使資源關閉或停止 (解除配置) 的情況下，雲端服務的 IP 位址也會保持相同。 此外，您可以轉換現有的動態 IP，作為保留的 IP 位址的 VIP。

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種：[Resource Manager 和傳統](../azure-resource-manager/resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 了解如何使用 [Resource Manager 部署模型](virtual-network-ip-addresses-overview-arm.md)來保留靜態公用 IP 位址。

若要深入了解 Azure 中的 IP 位址，請閱讀 [IP 位址](virtual-network-ip-addresses-overview-classic.md)文章。

## <a name="when-do-i-need-a-reserved-ip"></a>何時需要保留的 IP？
* **您想要確保 IP 會保留在您的訂用帳戶中**。 如果您想要保留一個在任何情況下都不會從您訂用帳戶釋出的 IP 位址，您應該使用保留的公用 IP。  
* **即使在已停止或解除配置狀態 (VM)，您想要保持 IP 與雲端服務之間的關聯**。 如果您想要讓使用者使用一個即使雲端服務中的 VM 被關閉或停止 (解除配置) 也不會變更的 IP 位址來存取服務。
* **您想要確保來自 Azure 的輸出流量使用可預測的 IP 位址**。 您可能必須設定內部部署防火牆，以便僅允許來自特定 IP 位址的流量。 藉由保留 IP，您會知道來源 IP 位址，而不必因為 IP 變更而需要更新您的防火牆規則。

## <a name="faq"></a>常見問題集
1. 我是否可以針對所有 Azure 服務都使用保留的 IP？ <br>
    編號 保留的 IP 僅可用於 VM 和雲端服務透過 VIP 公開的執行個體角色。
2. 我可以有多少保留的 IP？ <br>
    如需詳細資訊，請參閱 [Azure 限制](../azure-subscription-service-limits.md#networking-limits)一文。
3. 保留的 IP 是否會收取費用？ <br>
    有時是。 如需定價詳細資料，請參閱[保留的 IP 位址定價詳細資料](http://go.microsoft.com/fwlink/?LinkID=398482)頁面。
4. 我該如何保留 IP 位址？ <br>
    您可以使用 PowerShell、[Azure 管理 REST API](https://msdn.microsoft.com/library/azure/dn722420.aspx) 或 [Azure 入口網站](https://portal.azure.com)，在 Azure 區域中保留 IP 位址。 保留的 IP 位址會與您的訂用帳戶關聯。
5. 我是否可以將保留的 IP 位址與同質群組型 VNet 搭配使用？ <br>
    編號 保留的 IP 僅在區域 VNet 才受支援。 與同質群組關聯的 VNet 不支援保留的 IP。 如需有關將 VNet 與區域或同質群組建立關聯的詳細資訊，請參閱[關於區域 VNet 與同質群組](virtual-networks-migrate-to-regional-vnet.md)一文。

## <a name="manage-reserved-vips"></a>管理保留的 VIP

確保您完成[安裝並設定 PowerShell](/powershell/azure/overview) 文章中的步驟來安裝和設定 PowerShell。 

您必須將保留的 IP 新增至訂用帳戶才能使用。 若要在 *美國中部* 位置從可用的公用 IP 位址集區來建立保留的 IP，請執行下列命令：

```powershell
New-AzureReservedIP –ReservedIPName MyReservedIP –Location "Central US"
```

但是請注意，您無法指定正在保留的 IP。 若要檢視哪些 IP 位址會保留在訂用帳戶中，執行下列 PowerShell 命令，並注意 *ReservedIPName* 和 *Address* 的值：

```powershell
Get-AzureReservedIP
```

預期的輸出：

    ReservedIPName       : MyReservedIP
    Address              : 23.101.114.211
    Id                   : d73be9dd-db12-4b5e-98c8-bc62e7c42041
    Label                :
    Location             : Central US
    State                : Created
    InUse                : False
    ServiceName          :
    DeploymentName       :
    OperationDescription : Get-AzureReservedIP
    OperationId          : 55e4f245-82e4-9c66-9bd8-273e815ce30a
    OperationStatus      : Succeeded

>[!NOTE]
>使用 PowerShell 來建立保留的 IP 位址時，您無法指定資源群組以在其中建立保留的 IP。 Azure 會自動將它放在名為 *Default-Networking* 的資源群組中。 如果您使用 [Azure 入口網站](http://portal.azure.com)來建立保留的 IP，則可以指定您選擇的任何資源群組。 不過，如果您是在 *Default-Networking* 以外的資源群組中建立保留的 IP，則每當您使用 `Get-AzureReservedIP` 和 `Remove-AzureReservedIP` 之類的命令來參考保留的 IP 時，都必須參考 *Group resource-group-name reserved-ip-name* 名稱。  例如，如果您在名為 *myResourceGroup* 的資源群組中建立名為 *myReservedIP* 的保留 IP，就必須以 *Group myResourceGroup myReservedIP* 的形式參考保留的 IP 名稱。   

一旦保留 IP，其就會與您的訂用帳戶相關聯，直到刪除為止。 若要刪除保留的 IP，請執行下列 PowerShell 命令：

```powershell
Remove-AzureReservedIP -ReservedIPName "MyReservedIP"
```

## <a name="reserve-the-ip-address-of-an-existing-cloud-service"></a>保留現有雲端服務的 IP 位址
您可以新增 `-ServiceName` 參數，以保留現有雲端服務的 IP 位址。 若要在*美國中部*位置保留雲端服務 *TestService* 的 IP 位址，請執行下列 PowerShell 命令：

```powershell
New-AzureReservedIP –ReservedIPName MyReservedIP –Location "Central US" -ServiceName TestService
```

## <a name="associate-a-reserved-ip-to-a-new-cloud-service"></a>建立保留的 IP 至新雲端服務的關聯
下列指令碼會建立新的保留 IP，然後將它與名為 *TestService* 的新雲端服務建立關聯。

```powershell
New-AzureReservedIP –ReservedIPName MyReservedIP –Location "Central US"

$image = Get-AzureVMImage|?{$_.ImageName -like "*RightImage-Windows-2012R2-x64*"}

New-AzureVMConfig -Name TestVM -InstanceSize Small -ImageName $image.ImageName `
| Add-AzureProvisioningConfig -Windows -AdminUsername adminuser -Password MyP@ssw0rd!! `
| New-AzureVM -ServiceName TestService -ReservedIPName MyReservedIP -Location "Central US"
```

> [!NOTE]
> 當您建立保留的 IP 以與雲端服務搭配使用時，仍需使用 *VIP:&lt;連接埠號碼>* 來參照 VM 以進行輸入通訊。 保留 IP 並不表示您可以直接連接至 VM。 保留的 IP 會指派給已部署 VM 的雲端服務。 如果您想要透過 IP 直接連接到 VM，您必須設定執行個體層級公用 IP。 執行個體層級公用 IP 是一種直接指派給您 VM 的公用 IP (稱為 ILPIP)。 此類型 IP 無法保留。 如需詳細資訊，請參閱[執行個體層級公用 IP (ILPIP)](virtual-networks-instance-level-public-ip.md) 一文。
> 

## <a name="remove-a-reserved-ip-from-a-running-deployment"></a>從執行中部署移除保留的 IP
若要將已新增到新雲端服務的保留 IP 移除，請執行下列 PowerShell 命令：

```powershell
Remove-AzureReservedIPAssociation -ReservedIPName MyReservedIP -ServiceName TestService
```

> [!NOTE]
> 從執行中部署移除保留的 IP 並不會從您的訂用帳戶移除保留項目。 這僅會釋出 IP，以便訂用帳戶中的其他資源可以使用。
> 

## <a name="associate-a-reserved-ip-to-a-running-deployment"></a>建立保留的 IP 至執行中部署的關聯
下列命令會建立一個名為 *TestService2* 且具有名為 *TestVM2* 之新 VM 的雲端服務。 現有名為 *MyReservedIP* 的保留 IP 會接著與雲端服務建立關聯。

```powershell
$image = Get-AzureVMImage|?{$_.ImageName -like "*RightImage-Windows-2012R2-x64*"}

New-AzureVMConfig -Name TestVM2 -InstanceSize Small -ImageName $image.ImageName `
| Add-AzureProvisioningConfig -Windows -AdminUsername adminuser -Password MyP@ssw0rd!! `
| New-AzureVM -ServiceName TestService2 -Location "Central US"

Set-AzureReservedIPAssociation -ReservedIPName MyReservedIP -ServiceName TestService2
```

## <a name="associate-a-reserved-ip-to-a-cloud-service-by-using-a-service-configuration-file"></a>使用服務組態檔建立保留的 IP 至雲端服務的關聯
您也可以使用服務組態 (CSCFG) 檔建立保留的 IP 至雲端服務的關聯。 下列範例 XML 示範如何將雲端服務設定成使用名為 *MyReservedIP* 的保留 VIP：

    <?xml version="1.0" encoding="utf-8"?>
    <ServiceConfiguration serviceName="ReservedIPSample" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="4" osVersion="*" schemaVersion="2014-01.2.3">
      <Role name="WebRole1">
        <Instances count="1" />
        <ConfigurationSettings>
          <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="UseDevelopmentStorage=true" />
        </ConfigurationSettings>
      </Role>
      <NetworkConfiguration>
        <AddressAssignments>
          <ReservedIPs>
           <ReservedIP name="MyReservedIP"/>
          </ReservedIPs>
        </AddressAssignments>
      </NetworkConfiguration>
    </ServiceConfiguration>

## <a name="next-steps"></a>後續步驟
* 了解 [IP 位址](virtual-network-ip-addresses-overview-classic.md) 在傳統部署模型中的運作方式。
* 深入了解 [保留的私人 IP 位址](virtual-networks-reserved-private-ip.md)。
* 深入了解 [執行個體層級公用 IP (ILPIP) 位址](virtual-networks-instance-level-public-ip.md)。

