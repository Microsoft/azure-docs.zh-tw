---
title: Azure 虛擬網路服務端點原則 | Microsoft Docs
description: 了解如何使用服務端點原則篩選流向 Azure 服務資源的虛擬網路流量
services: virtual-network
documentationcenter: na
author: RDhillon
ms.service: virtual-network
ms.devlang: NA
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/21/2020
ms.author: rdhillon
ms.openlocfilehash: 9766379807e6d2708fd6935dd2ffbd7660f9988f
ms.sourcegitcommit: d59abc5bfad604909a107d05c5dc1b9a193214a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98216643"
---
# <a name="virtual-network-service-endpoint-policies-for-azure-storage"></a>Azure 儲存體的虛擬網路服務端點原則

虛擬網路 (VNet) 服務端點原則可讓您透過服務端點，將輸出虛擬網路流量篩選到 Azure 儲存體帳戶，並只允許將資料遭到外泄至特定的 Azure 儲存體帳戶。 端點原則可在透過服務端點連接時，為虛擬網路流量提供更細微的存取控制，以 Azure 儲存體。

![保護 Azure 儲存體帳戶的虛擬網路輸出流量](./media/virtual-network-service-endpoint-policies-overview/vnet-service-endpoint-policies-overview.png)

這項功能已在 __全球所有 Azure 區域__ 中正式推出 __Azure 儲存體__。

## <a name="key-benefits"></a>主要權益

虛擬網路服務端點原則具有下列優勢：

- __提升虛擬網路流量的安全性以 Azure 儲存體__

  [適用于網路安全性群組的 Azure 服務](./network-security-groups-overview.md) 標籤可讓您將虛擬網路輸出流量限制為特定的 Azure 儲存體區域。 不過，這可允許所選 Azure 儲存體區域內任何帳戶的流量。
  
  端點原則可讓您指定允許虛擬網路輸出存取的 Azure 儲存體帳戶，並限制所有其他儲存體帳戶的存取權。 這可提供更細微的安全性控制，以保護從虛擬網路遭到外泄的資料。

- __可用來篩選 Azure 服務流量的可調整與高可用性原則__

   端點原則提供可水平調整的高可用性解決方案，以篩選從虛擬網路透過服務端點通往 Azure 服務的流量。 在虛擬網路中，這個流量完全不會對中央網路設備的維護產生額外負荷。

## <a name="json-object-for-service-endpoint-policies"></a>服務端點原則的 JSON 物件
讓我們快速查看服務端點原則物件。

```json
"serviceEndpointPolicyDefinitions": [
    {
            "description": null,
            "name": "MySEP-Definition",
            "resourceGroup": "MySEPDeployment",
            "service": "Microsoft.Storage",
            "serviceResources": [ 
                    "/subscriptions/subscriptionID/resourceGroups/MySEPDeployment/providers/Microsoft.Storage/storageAccounts/mystgacc"
            ],
            "type": "Microsoft.Network/serviceEndpointPolicies/serviceEndpointPolicyDefinitions"
    }
]
```

## <a name="configuration"></a>組態

-   您可以設定端點原則來限制特定 Azure 儲存體帳戶的虛擬網路流量。
-   您可在虛擬網路的子網路上設定端點原則。 您應在子網上啟用 Azure 儲存體的服務端點，以套用原則。
-   端點原則可讓您使用 resourceID 格式，將特定 Azure 儲存體帳戶新增至允許清單。 您可以限制存取
    - 訂用帳戶中的所有儲存體帳戶<br>
      `E.g. /subscriptions/subscriptionId`

    - 資源群組中的所有儲存體帳戶<br>
      `E.g. subscriptions/subscriptionId/resourceGroups/resourceGroupName`
     
    - 個別的儲存體帳戶，其方式是列出對應的 Azure Resource Manager resourceId。 這包含 Blob、資料表、佇列、檔案和 Azure Data Lake Storage Gen2 的流量。 <br>
    `E.g. /subscriptions/subscriptionId/resourceGroups/resourceGroupName/providers/Microsoft.Storage/storageAccounts/storageAccountName`
-   根據預設，如果沒有任何原則附加至具有端點的子網，您可以存取服務中的所有儲存體帳戶。 一旦在這個子網路上設定原則之後，就只能從該子網路的計算執行個體存取原則中所指定資源。 將會拒絕存取所有其他儲存體帳戶。
-   在子網上套用服務端點原則時，Azure 儲存體 *服務端點範圍* 會從區域升級為 **全域**。 這表示，Azure 儲存體的所有流量都會在之後透過服務端點來保護。 服務端點原則也適用于全域，因此任何未明確允許的儲存體帳戶都將被拒絕存取。 
-   您可以將多個原則套用至子網路。 當有多個原則與子網建立關聯時，將允許虛擬網路流量流向這些原則之間指定的資源。 若要存取任何原則中未指定的所有其他服務資源，則會遭到拒絕。

    > [!NOTE]  
    > 服務端點原則 **允許原則**，除了指定的資源之外，其他所有資源都會受到限制。 請確定應用程式的所有服務資源相依性都已識別並列于原則中。

- 端點原則中只能指定使用 Azure 資源模型的儲存體帳戶。 您的傳統 Azure 儲存體帳戶將不支援 Azure 服務端點原則。
- 如果已列出主要帳戶，系統就會自動允許 RA-GRS 次要存取。
- 儲存體帳戶不一定要位於與虛擬網路相同的訂用帳戶或 Azure Active Directory 租用戶中。

## <a name="scenarios"></a>案例

- **對等互連或多個虛擬網路**：若要篩選對等互連虛擬網路中的流量，您應該將端點原則個別套用至這些虛擬網路。
- **使用網路設備或 Azure 防火牆篩選網際網路流量**：使用原則、透過服務端點來篩選 Azure 服務流量，並透過設備或 azure 防火牆來篩選其餘網際網路或 azure 流量。
- **篩選部署到虛擬網路的 azure 服務流量**：目前，部署至虛擬網路中的任何受控 Azure 服務都不支援 Azure 服務端點原則。 
- **篩選從內部部署到 Azure 服務的流量**：服務端點原則只適用於來自原則相關聯子網路的流量。 若要允許從內部部署存取特定的 Azure 服務資源，則應該使用網路虛擬設備或防火牆篩選流量。

## <a name="logging-and-troubleshooting"></a>記錄和疑難排解
服務端點原則並未提供任何集中式記錄。 針對服務資源記錄，請參閱 [服務端點記錄](virtual-network-service-endpoints-overview.md#logging-and-troubleshooting)。

### <a name="troubleshooting-scenarios"></a>疑難排解案例
- 存取處於預覽狀態的儲存體帳戶， (不在地理配對區域中的存取權) 
  - Azure 儲存體升級為使用全域服務標籤，則服務端點的範圍和服務端點原則現在是全域的。 因此，對 Azure 儲存體的任何流量都會透過服務端點進行加密，而只有在原則中明確列出的儲存體帳戶才能存取。
  - 明確允許列出所有必要的儲存體帳戶，以還原存取權。  
  - 連絡 Azure 支援。
- 存取端點原則中所列的帳戶時遭到拒絕
  - 網路安全性群組或防火牆篩選條件可能會封鎖存取
  - 如果移除/重新套用原則會導致連線中斷：
    - 驗證 Azure 服務是否設定為允許透過端點從虛擬網路存取，或資源的預設原則設定為 [ *全部允許*]。
    - 驗證服務診斷是否顯示透過端點的流量。
    - 請檢查網路安全性群組流量記錄是否顯示存取狀態，而儲存體記錄是否如預期般顯示透過服務端點的存取狀態。
    - 連絡 Azure 支援。
- 存取服務端點原則中未列的帳戶時遭到拒絕
  - 驗證 Azure 儲存體是否設定為允許透過端點從虛擬網路存取，或資源的預設原則是否設定為 [ *全部允許*]。
  - 確定帳戶不是 **傳統的儲存體帳戶** ，且子網上具有服務端點原則。
- 在子網上套用服務端點原則之後，受控 Azure 服務已停止運作
  - 目前服務端點原則不支援受控服務。 請 *觀賞此空間以取得更新*。

## <a name="provisioning"></a>佈建

擁有虛擬網路寫入權的使用者可以在子網路上設定服務端點原則。 深入了解 Azure [內建角色](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)，以及如何將特定權限指派給[自訂角色](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

虛擬網路和 Azure 儲存體帳戶可以位於相同或不同的訂用帳戶，或 Azure Active Directory 租使用者中。

## <a name="limitations"></a>限制

- 您只能在透過 Azure Resource Manager 部署模型所部署的虛擬網路上部署服務端點原則。
- 虛擬網路必須位於與服務端點原則相同的區域中。
- 如果原則中列出的 Azure 服務已設定服務端點，您就只能將服務端點原則套用至子網路。
- 如果流量是從內部部署網路流向 Azure 服務，您就無法對其使用服務端點原則。
- Azure 受控服務目前不支援端點原則。 這包括部署到共用子網的受控服務 (例如 *Azure Batch、AZURE 新增、Azure 應用程式閘道、AZURE VPN 閘道、Azure 防火牆*) 或專用子網 (例如 *Azure App Service 環境、azure Redis 快取、azure API 管理、azure SQL MI、傳統受控服務*) 。

 > [!WARNING]
 > 針對基礎結構需求，部署到虛擬網路中的 Azure 服務 (例如 Azure HDInsight) 可存取其他 Azure 服務 (例如 Azure 儲存體)。 如果您將端點原則限制給特定資源，可能會中斷存取虛擬網路中已部署 Azure 服務的這些基礎結構資源。

- 端點原則不支援傳統儲存體帳戶。 原則預設會拒絕存取所有傳統儲存體帳戶。 如果您的應用程式需要存取 Azure Resource Manager 和傳統儲存體帳戶，就不應針對此流量使用端點原則。

## <a name="pricing-and-limits"></a>價格和限制

使用服務端點原則不會額外收費。 透過服務端點的 Azure 服務 (例如 Azure 儲存體) 其目前定價模式照常適用。

服務端點原則會強制執行下列限制： 

 |資源 | 預設限制 |
 |---------|---------------|
 |ServiceEndpointPoliciesPerSubscription |500 |
 |ServiceEndpointPoliciesPerSubnet|100 |
 |ServiceResourcesPerServiceEndpointPolicyDefinition|200 |

## <a name="next-steps"></a>後續步驟

- 了解[如何設定虛擬網路服務端點原則](virtual-network-service-endpoint-policies-portal.md)
- 深入瞭解 [虛擬網路服務端點](virtual-network-service-endpoints-overview.md)