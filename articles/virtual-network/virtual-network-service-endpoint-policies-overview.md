---
title: Azure 虛擬網路服務端點原則 | Microsoft Docs
description: 了解如何使用服務端點原則篩選流向 Azure 服務資源的虛擬網路流量
services: virtual-network
documentationcenter: na
author: anithaa
manager: narayan
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/18/2018
ms.author: anithaa
ms.custom: ''
ms.openlocfilehash: 425bbc9eac112a4b999bd08940abb8b875aca61c
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47433288"
---
# <a name="virtual-network-service-endpoint-policies-preview"></a>虛擬網路服務端點原則 (預覽)

虛擬網路 (VNet) 服務端點原則可讓您篩選流向 Azure 服務的虛擬網路流量，以僅允許特定 Azure 服務資源經過服務端點。 端點原則可針對流向 Azure 服務的虛擬網路流量提供更細微的存取控制。

這項功能會以__預覽__形式在下列 Azure 服務和地區提供：

__Azure 儲存體__：WestCentralUS、WestUS2。

如需此預覽的最新通知，請參閱 [Azure 虛擬網路更新](https://azure.microsoft.com/updates/?product=virtual-network)頁面。

> [!NOTE]  
> 在預覽階段，虛擬網路服務端點原則的可用性和可靠性可能無法達到正式運作版功能的水準。 如需詳細資訊，請參閱 [Microsoft Azure 預覽版增補使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="key-benefits"></a>主要權益

虛擬網路服務端點原則具有下列優勢：

- __提升流向 Azure 服務的虛擬網路流量安全性__

  [網路安全性群組的 Azure 服務標籤](https://aka.ms/servicetags)可讓您將虛擬網路輸出流量限制為特定 Azure 服務。 不過，這會允許流向該 Azure 服務任何資源的流量。 
  
  現在，您可以使用端點原則，將虛擬網路輸出存取限制為僅限特定 Azure 資源。 這可提供更細微的安全性控制，以保護在您虛擬網路中受存取的資料。 

- __可用來篩選 Azure 服務流量的可調整與高可用性原則__

   端點原則提供可水平調整的高可用性解決方案，以篩選從虛擬網路透過服務端點通往 Azure 服務的流量。 在虛擬網路中，這個流量完全不會對中央網路設備的維護產生額外負荷。

## <a name="configuration"></a>組態

- 您可以設定端點原則，將虛擬網路流量限制為特定 Azure 服務資源。 針對預覽版本，我們支援 Azure 儲存體的端點原則。 
- 您可在虛擬網路的子網路上設定端點原則。 針對原則中列出的所有 Azure 服務，您應該在子網路上啟用服務端點以套用原則。
- 端點原則可讓您使用 resourceID 格式，將特定 Azure 服務資源列入白名單。 您可以限制訂用帳戶或資源群組中所有資源的存取。 您也可以限制特定資源的存取。 
- 根據預設，如果含端點的子網路未附加任何原則，您就可以存取服務中的所有資源。 一旦在這個子網路上設定原則之後，就只能從該子網路的計算執行個體存取原則中所指定資源。 若要存取特定服務的其他所有資源，則會遭到拒絕。 
- 您可以篩選流向服務端點所設區域中的 Azure 服務資源流量。 預設會允許存取其他區域的服務資源。 

  > [!NOTE]  
  > 若要將虛擬網路輸出存取完全鎖定在服務端點區域中的 Azure 服務資源，您也需要使用[服務標籤](security-overview.md#service-tags)，將網路安全性群組設定為僅允許流向服務端點區域的流量。

- 您可以將多個原則套用至子網路。 當多個原則都與相同服務的子網路建立關聯時，即會允許虛擬網路流量流向任何這些原則所指定的資源。 若要存取任何原則中未指定的所有其他服務資源，則會遭到拒絕。 

  > [!NOTE]  
  > 原則僅允許從虛擬網路存取列出的服務資源。 若您將特定資源新增至原則，系統會自動拒絕流向服務的所有其他流量。 請確認原則中已列出並可識別應用程式的所有服務資源相依性。

  > [!WARNING]  
  > 針對基礎結構需求，部署到虛擬網路中的 Azure 服務 (例如 Azure HDInsight) 可存取其他 Azure 服務 (例如 Azure 儲存體)。 如果您將端點原則限制給特定的資源，可能會中斷存取虛擬網路中已部署服務的這些基礎結構資源。 如需了解特定的服務，請參閱[限制](#limitations)。在預覽期間，服務端點原則不支援部署於虛擬網路中的任何受控 Azure 服務。

- 針對 Azure 儲存體： 
  -  您可以列出儲存體帳戶的 Azure Resource Manager *resourceId* 以限制存取權。 這包含 Blob、資料表、佇列、檔案和 Azure Data Lake Storage Gen2 的流量。

     例如，端點原則定義中所列的 Azure 儲存體帳戶可能如下所示：
    
     若要允許特定的儲存體帳戶：         
     `subscriptions/subscriptionId/resourceGroups/resourceGroup/providers/Microsoft.Storage/storageAccounts/storageAccountName`
      
     若要允許訂用帳戶和資源群組中的所有帳戶：`/subscriptions/subscriptionId/resourceGroups/resourceGroup`
     
     若要允許訂用帳戶中的所有帳戶：`/subscriptions/subscriptionId`
    
- 端點原則中只能指定使用 Azure 資源模型的儲存體帳戶。  

  > [!NOTE]  
  > 端點原則會封鎖傳統儲存體帳戶的存取。

- 針對子網路，所列出帳戶的主要位置應該在服務端點地理配對區域中。 

  > [!NOTE]  
  > 原則可讓您指定其他區域的服務資源。 系統只會依據地理配對區域來篩選虛擬網路對 Azure 服務的存取。 如果您未將網路安全性群組限制為 Azure 儲存體的地理配對區域，虛擬網路就可以存取地理配對區域外的所有儲存體帳戶。

- 如果已列出主要帳戶，系統就會自動允許 RA-GRS 次要存取。 
- 儲存體帳戶不一定要位於與虛擬網路相同的訂用帳戶或 Azure Active Directory 租用戶中。 

## <a name="limitations"></a>限制

- 您只能在透過 Azure Resource Manager 部署模型所部署的虛擬網路上部署服務端點原則。
- 虛擬網路必須位於與服務端點原則相同的區域中。
- 如果原則中列出的 Azure 服務已設定服務端點，您就只能將服務端點原則套用至子網路。
- 如果流量是從內部部署網路流向 Azure 服務，您就無法對其使用服務端點原則。
- 如果子網路具有受控的 Azure 服務且基礎結構需求上對 Azure 服務有相依性，您就不應該將端點原則套用至這些子網路。 

  > [!WARNING]  
  > 針對基礎結構需求，部署到虛擬網路中的 Azure 服務 (例如 Azure HDInsight) 可存取其他 Azure 服務 (例如 Azure 儲存體)。 如果您將端點原則限制給特定資源，可能會中斷存取虛擬網路中已部署 Azure 服務的這些基礎結構資源。
  
  - 某些 Azure 服務可以部署到具有其他計算執行個體的子網路中。 如果子網路部署了以下所列的受控服務，請不要將端點原則套用至子網路。
   
    - Azure HDInsight
    - Azure Batch (Azure Resource Manager)
    - Azure Active Directory Domain Services (Azure Resource Manager)
    - Azure 應用程式閘道 (Azure Resource Manager)
    - Azure VPN 閘道 (Azure Resource Manager)
    - Azure 防火牆

  - 某些 Azure 服務是部署在專用的子網路中。 在預覽期間，以下列出的這類所有服務都會封鎖端點原則。 

     - Azure App Service Environment
     - Azure Rediscache
     - Azure API 管理
     - Azure SQL 受控執行個體
     - Azure Active Directory Domain Services
     - Azure 應用程式閘道 (傳統)
     - Azure VPN 閘道 (傳統)

- Azure 儲存體：端點原則不支援傳統儲存體帳戶。 原則預設會拒絕存取所有傳統儲存體帳戶。 如果您的應用程式需要存取 Azure Resource Manager 和傳統儲存體帳戶，就不應針對此流量使用端點原則。 

## <a name="nsgs-with-service-endpoint-policies"></a>使用服務端點原則的 NSG
- 根據預設，NSG 允許輸出網際網路流量，包括流向 Azure 服務的虛擬網路流量。
- 如果您要拒絕所有輸出網際網路流量，只允許對特定 Azure 服務資源的流量： 

  步驟 1：使用 *Azure 服務標籤*，將 NSG 設定為只允許流向端點區域中 Azure 服務的輸出流量。 如需詳細資訊，請參閱 [NSG 服務標籤](https://aka.ms/servicetags)
      
  例如，僅限存取端點區域的網路安全性群組規則看起來會如下範例所示：

  ```
  Allow AzureStorage.WestUS2,
  Allow AzureStorage.WestCentralUS,
  Deny all
  ```

  步驟 2：將服務端點原則與存取權套用至特定的 Azure 服務資源。

  > [!WARNING]  
  > 如果您未將網路安全性群組設定為僅限虛擬網路的 Azure 服務存取端點區域，那麼即使套用服務端點原則，您仍可以存取位於其他區域中的服務資源。

## <a name="scenarios"></a>案例

- **對等互連或多個虛擬網路**：若要篩選對等互連虛擬網路中的流量，您應該將端點原則個別套用至這些虛擬網路。
- **使用網路設備或 Azure 防火牆篩選網際網路流量**：使用原則來篩選通過端點的 Azure 服務流量，並透過設備或 Azure 防火牆來篩選其餘網際網路或 Azure 流量。 
- **篩選部署於虛擬網路中 Azure 服務的流量**：在預覽期間，服務端點原則不支援部署於虛擬網路中的任何受控 Azure 服務。 
 如需特定服務，請參閱[限制](#Limitations)。
- **篩選從內部部署到 Azure 服務的流量**：服務端點原則只適用於來自原則相關聯子網路的流量。 若要允許從內部部署存取特定的 Azure 服務資源，則應該使用網路虛擬設備或防火牆篩選流量。

## <a name="logging-and-troubleshooting"></a>記錄和疑難排解
服務端點原則並未提供任何集中式記錄。 如需服務診斷記錄，請參閱[服務端點記錄](virtual-network-service-endpoints-overview.md#logging-and-troubleshooting)。

### <a name="troubleshooting-scenarios"></a>疑難排解案例
- 端點原則未列出允許存取的儲存體帳戶
  - 網路安全性群組可能會允許存取網際網路或位於其他區域的 Azure 儲存體帳戶。
  - 如果要拒絕所有輸出網際網路流量，並只允許特定 Azure 儲存體區域的流量，則應該設定網路安全性群組。 如需詳細資料，請參閱[網路安全性群組](#network-security-groups)。
- 存取端點原則中所列的帳戶時遭到拒絕
  - 網路安全性群組或防火牆篩選條件可能會封鎖存取
  - 如果移除/重新套用原則會導致連線中斷：
   - 驗證 Azure 服務是否設定為允許透過端點從虛擬網路存取，或資源的預設原則是否設定為「全部允許」。
      > [!NOTE]      
      > 服務資源不需要在虛擬網路中受到保護，也可以透過端點原則取得存取權。 不過，我們建議的安全性最佳做法是使用受信任網路來保護服務資源，例如：若是 Azure 虛擬網路，請透過服務端點來保護；若是內部部署，則透過 IP 防火牆來保護。
  
   - 驗證服務診斷是否顯示透過端點的流量。
    - 請檢查網路安全性群組流量記錄是否顯示存取狀態，而儲存體記錄是否如預期般顯示透過服務端點的存取狀態。
    - 連絡 Azure 支援。
- 存取服務端點原則中未列的帳戶時遭到拒絕
  - 網路安全性群組或防火牆篩選條件可能會封鎖存取。 請確認端點區域允許使用「Azure 儲存體」服務標籤。 如需了解原則的限制，請參閱[限制](#limitations)。
  例如，如果套用原則，則會拒絕傳統儲存體帳戶的存取。
  - 驗證 Azure 服務是否設定為允許透過端點從虛擬網路存取，或資源的預設原則是否設定為「全部允許」。

## <a name="provisioning"></a>佈建

擁有虛擬網路寫入權的使用者可以在子網路上設定服務端點原則。 深入了解 Azure [內建角色](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)，以及如何將特定權限指派給[自訂角色](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

虛擬網路和 Azure 服務資源不一定要位於相同的訂用帳戶或 Azure Active Directory 租用戶中。 

## <a name="pricing-and-limits"></a>價格和限制

使用服務端點原則不會額外收費。 透過服務端點的 Azure 服務 (例如 Azure 儲存體) 其目前定價模式照常適用。

服務端點原則會強制執行下列限制： 

 |資源 | 預設限制 |
 |---------|---------------|
 |ServiceEndpointPoliciesPerSubscription |500 |
 |ServiceEndpintPoliciesPerSubnet|100 |
 |ServiceResourcesPerServiceEndpointPolicyDefinition|200 |

## <a name="next-steps"></a>後續步驟

- 了解[如何設定虛擬網路服務端點原則](virtual-network-service-endpoint-policies-portal.md)
- 深入了解[虛擬網路服務端點](virtual-network-service-endpoints-overview.md)

