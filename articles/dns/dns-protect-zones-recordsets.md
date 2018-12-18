---
title: 保護 DNS 區域和記錄 | Microsoft Docs
description: 如何在 Microsoft Azure DNS 中保護 DNS 區域和記錄集。
services: dns
documentationcenter: na
author: vhorne
manager: jeconnoc
ms.assetid: 190e69eb-e820-4fc8-8e9a-baaf0b3fb74a
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/20/2016
ms.author: victorh
ms.openlocfilehash: ff20c16c89ca5bf27bfddc654119b428cc425d2d
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39173462"
---
# <a name="how-to-protect-dns-zones-and-records"></a>如何保護 DNS 區域和記錄

DNS 區域和記錄是重要的資源。 刪除 DNS 區域或甚至只是單一的 DNS 記錄，可能會導致整個服務中斷。  因此，保護重要的 DNS 區域和記錄以防未經授權或意外變更相當重要。

這篇文章說明 Azure DNS 如何讓您保護您的 DNS 區域和記錄免於這類變更。  我們會套用 Azure Resource Manager 提供的兩個強大的安全性功能︰[角色型存取控制](../role-based-access-control/overview.md)和[資源鎖定](../azure-resource-manager/resource-group-lock-resources.md)。

## <a name="role-based-access-control"></a>角色型存取控制

Azure 角色型存取控制 (RBAC) 可以對 Azure 使用者、群組和資源進行更細緻的存取權管理。 使用 RBAC，您可以準確地授與使用者執行其作業所需的存取權。 如需有關 RBAC 如何協助您管理存取權的詳細資訊，請參閱[什麼是角色型存取控制](../role-based-access-control/overview.md)。

### <a name="the-dns-zone-contributor-role"></a>「DNS 區域參與者」角色

「DNS 區域參與者」角色是內建角色，由 Azure 提供用於管理 DNS 資源。  將「DNS 區域參與者」權限指派給使用者或群組，可讓該群組管理 DNS 資源，但是無法管理任何其他類型的資源。

例如，假設資源群組 'myzones' 包含 Contoso Corporation 的五個區域。 將該資源群組的「DNS 區域參與者」權限授與 DNS 系統管理員，可讓他們具有這些 DNS 區域的完整控制權。 它也可避免授與不必要的權限，例如 DNS 系統管理員無法建立或停止虛擬機器。

指派 RBAC 權限的最簡單方式是[透過 Azure 入口網站](../role-based-access-control/role-assignments-portal.md)。  開啟資源群組的 [存取控制 (IAM)] 刀鋒視窗，按一下 [新增]、選取 [DNS 區域參與者] 角色，然後選取必要的使用者或群組以授與權限。

![透過 Azure 入口網站的資源群組層級 RBAC](./media/dns-protect-zones-recordsets/rbac1.png)

權限也可以[使用 Azure PowerShell 授與](../role-based-access-control/role-assignments-powershell.md)：

```powershell
# Grant 'DNS Zone Contributor' permissions to all zones in a resource group
New-AzureRmRoleAssignment -SignInName "<user email address>" -RoleDefinitionName "DNS Zone Contributor" -ResourceGroupName "<resource group name>"
```

對等的命令也可以[透過 Azure CLI 使用](../role-based-access-control/role-assignments-cli.md)：

```azurecli
# Grant 'DNS Zone Contributor' permissions to all zones in a resource group
azure role assignment create --signInName "<user email address>" --roleName "DNS Zone Contributor" --resourceGroup "<resource group name>"
```

### <a name="zone-level-rbac"></a>區域層級 RBAC

Azure RBAC 規則可以套用至訂用帳戶、資源群組或個別資源。 在 Azure DNS 的情況下，該資源可以是個別 DNS 區域或甚至是個別記錄集。

例如，假設資源群組 'myzones' 包含區域 'contoso.com' 和子區域 'customers.contoso.com'，其中 CNAME 記錄是針對每個客戶帳戶建立的。  用來管理這些 CNAME 記錄的帳戶應該獲得指派權限以僅在 'customers.contoso.com' 區域中建立記錄，它不應該有其他區域的存取權。

區域層級 RBAC 權限可以透過 Azure 入口網站授與。  開啟區域的 [存取控制 (IAM)] 刀鋒視窗，按一下 [新增]、選取 [DNS 區域參與者] 角色，然後選取必要的使用者或群組以授與權限。

![透過 Azure 入口網站的 DNS 區域層級 RBAC](./media/dns-protect-zones-recordsets/rbac2.png)

權限也可以[使用 Azure PowerShell 授與](../role-based-access-control/role-assignments-powershell.md)：

```powershell
# Grant 'DNS Zone Contributor' permissions to a specific zone
New-AzureRmRoleAssignment -SignInName "<user email address>" -RoleDefinitionName "DNS Zone Contributor" -ResourceGroupName "<resource group name>" -ResourceName "<zone name>" -ResourceType Microsoft.Network/DNSZones
```

對等的命令也可以[透過 Azure CLI 使用](../role-based-access-control/role-assignments-cli.md)：

```azurecli
# Grant 'DNS Zone Contributor' permissions to a specific zone
azure role assignment create --signInName <user email address> --roleName "DNS Zone Contributor" --resource-name <zone name> --resource-type Microsoft.Network/DNSZones --resource-group <resource group name>
```

### <a name="record-set-level-rbac"></a>記錄集層級 RBAC

我們可以繼續下一步。 請考慮 Contoso Corporation 的郵件系統管理員，他需要存取位於 'contoso.com' 區域頂點的 MX 和 TXT 記錄。  她不需要存取任何其他 MX 或 TXT 記錄，或任何其他類型的任何記錄。  Azure DNS 可讓您在記錄集層級指派權限，準確地給予郵件系統管理員需要存取的記錄。  郵件系統管理員準確地獲得她需要的控制權，且無法進行任何其他變更。

記錄集層級 RBAC 權限可透過 Azure 入口網站，使用記錄集刀鋒視窗中的 [使用者] 按鈕來設定︰

![透過 Azure 入口網站的記錄集層級 RBAC](./media/dns-protect-zones-recordsets/rbac3.png)

記錄集層級 RBAC 權限也可以[使用 Azure PowerShell 授與](../role-based-access-control/role-assignments-powershell.md)：

```powershell
# Grant permissions to a specific record set
New-AzureRmRoleAssignment -SignInName "<user email address>" -RoleDefinitionName "DNS Zone Contributor" -Scope "/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Network/dnszones/<zone name>/<record type>/<record name>"
```

對等的命令也可以[透過 Azure CLI 使用](../role-based-access-control/role-assignments-cli.md)：

```azurecli
# Grant permissions to a specific record set
azure role assignment create --signInName "<user email address>" --roleName "DNS Zone Contributor" --scope "/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Network/dnszones/<zone name>/<record type>/<record name>"
```

### <a name="custom-roles"></a>自訂角色

內建「DNS 區域參與者」角色可以有 DNS 資源的完整控制權。 此外，也可以建置自己的客戶 Azure 角色，以提供更細微的控制。

請再考量一次區域 'customers.contoso.com' 中的 CNAME 記錄是針對每個 Contoso Corporation 客戶帳戶建立的範例。  用來管理這些 CNAME 的帳戶應該授與只管理 CNAME 記錄的權限。  如此便無法修改其他類型的記錄 (例如變更 MX 記錄)，或執行區域層級作業，例如區域刪除。

下列範例示範只管理 CNAME 記錄的自訂角色定義︰

```json
{
    "Name": "DNS CNAME Contributor",
    "Id": "",
    "IsCustom": true,
    "Description": "Can manage DNS CNAME records only.",
    "Actions": [
        "Microsoft.Network/dnsZones/CNAME/*",
        "Microsoft.Network/dnsZones/read",
        "Microsoft.Authorization/*/read",
        "Microsoft.Insights/alertRules/*",
        "Microsoft.ResourceHealth/availabilityStatuses/read",
        "Microsoft.Resources/deployments/*",
        "Microsoft.Resources/subscriptions/resourceGroups/read",
        "Microsoft.Support/*"
    ],
    "NotActions": [
    ],
    "AssignableScopes": [
        "/subscriptions/ c276fc76-9cd4-44c9-99a7-4fd71546436e"
    ]
}
```

Actions 屬性會定義下列 DNS 特定權限︰

* `Microsoft.Network/dnsZones/CNAME/*` 授與 CNAME 記錄的完整控制權
* `Microsoft.Network/dnsZones/read` 授與權限以讀取 DNS 區域，但無法修改它們，讓您查看在其中建立 CNAME 的區域。

剩餘的動作會從 [DNS 區域參與者內建角色](../role-based-access-control/built-in-roles.md#dns-zone-contributor)複製。

> [!NOTE]
> 使用自訂 RBAC 角色，以避免刪除記錄集，同時仍然允許更新它們不是有效的控制。 它會防止記錄集被刪除，但它不會防止它們被修改。  允許的修改包括從記錄集新增和移除記錄，包括移除所有記錄以保持「空白」記錄集。 這與從 DNS 解析觀點來刪除記錄集有相同的效果。

目前無法透過 Azure 入口網站定義自訂角色定義。 可以使用 Azure PowerShell 建立根據此角色定義的自訂角色︰

```powershell
# Create new role definition based on input file
New-AzureRmRoleDefinition -InputFile <file path>
```

也可以透過 Azure CLI 建立︰

```azurecli
# Create new role definition based on input file
azure role create -inputfile <file path>
```

然後可以使用與內建角色相同的方式指派角色，如本文稍早所述。

如需有關如何建立、管理及指派自訂角色的詳細資訊，請參閱[在 Azure RBAC 中自訂角色](../role-based-access-control/custom-roles.md)。

## <a name="resource-locks"></a>資源鎖定

除了 RBAC，Azure Resource Manager 支援另一種安全性控制項，也就是「鎖定」資源的能力。 其中 RBAC 規則可讓您控制特定使用者和群組的動作，資源鎖定會套用至資源，而且對於所有使用者和角色都有效。 如需詳細資訊，請參閱[使用 Azure Resource Manager 來鎖定資源](../azure-resource-manager/resource-group-lock-resources.md)。

有兩種類型的資源鎖定︰**DoNotDelete** 和 **ReadOnly**。 這些可以套用至 DNS 區域，或個別的記錄集。  下列章節說明幾個常見的案例，以及如何使用資源鎖定進行支援。

### <a name="protecting-against-all-changes"></a>保護免於所有變更

若要避免任何變更，將 ReadOnly 鎖定套用至區域。  這樣可以防止建立新的記錄集，現有記錄集也能免於遭到修改或刪除。

可以透過 Azure 入口網站來建立區域層級資源鎖定。  從 DNS 區域刀鋒視窗中，按一下 [鎖定]，然後按一下 [新增]：

![透過 Azure 入口網站的區域層級資源鎖定](./media/dns-protect-zones-recordsets/locks1.png)

也可以透過 Azure PowerShell 來建立區域層級資源鎖定：

```powershell
# Lock a DNS zone
New-AzureRmResourceLock -LockLevel <lock level> -LockName <lock name> -ResourceName <zone name> -ResourceType Microsoft.Network/DNSZones -ResourceGroupName <resource group name>
```

設定 Azure 資源鎖定目前無法透過 Azure CLI 獲得支援。

### <a name="protecting-individual-records"></a>保護個別記錄

若要防止現有 DNS 記錄集遭到修改，請將 ReadOnly 鎖定套用至記錄集。

> [!NOTE]
> 將 DoNotDelete 鎖定套用至記錄集不是有效的控制。 它會防止記錄集被刪除，但它不會防止它被修改。  允許的修改包括從記錄集新增和移除記錄，包括移除所有記錄以保持「空白」記錄集。 這與從 DNS 解析觀點來刪除記錄集有相同的效果。

記錄集層級資源鎖定目前只能使用 Azure PowerShell 進行設定。  在 Azure 入口網站或 Azure CLI 不支援。

```powershell
# Lock a DNS record set
New-AzureRmResourceLock -LockLevel <lock level> -LockName "<lock name>" -ResourceName "<zone name>/<record set name>" -ResourceType "Microsoft.Network/DNSZones/<record type>" -ResourceGroupName "<resource group name>"
```

### <a name="protecting-against-zone-deletion"></a>防止區域刪除

在 Azure DNS 中刪除區域時，也會一併刪除區域中的所有記錄集。  此作業無法復原。  不小心刪除重要區域，可能會對業務造成嚴重影響。  因此，防止意外區域刪除非常重要。

將 DoNotDelete 鎖定套用至區域可防止區域刪除。  不過，因為鎖定會由子資源繼承，也會防止區域中任何記錄集遭到刪除，這可能不是想要的結果。  此外，如以上所述，因為記錄仍然可以從現有的記錄集移除，所以它是無效的。

另外，請考慮將 DoNotDelete 鎖定套用至區域中的記錄集，例如 SOA 記錄集。  因為在未刪除記錄集的情況下無法刪除區域，這樣可以防止區域刪除，同時讓區域內的記錄集可以自由地被修改。 如果嘗試刪除區域，Azure Resource Manager 會偵測到這樣也會刪除 SOA 記錄集，並且封鎖呼叫，因為 SOA 已被鎖定。  不會刪除任何記錄集。

下列 PowerShell 命令會針對指定區域的 SOA 記錄建立 DoNotDelete 鎖定︰

```powershell
# Protect against zone delete with DoNotDelete lock on the record set
New-AzureRmResourceLock -LockLevel DoNotDelete -LockName "<lock name>" -ResourceName "<zone name>/@" -ResourceType" Microsoft.Network/DNSZones/SOA" -ResourceGroupName "<resource group name>"
```

防止意外區域刪除的另一個方法是使用自訂的角色來確保用來管理您的區域的操作員和服務帳戶不具備區域刪除權限。 真的需要刪除區域時，您可以強制執行兩個步驟刪除，先授與區域刪除權限 (在區域範圍，以防止刪除錯誤的區域)，然後其次刪除區域。

這個第二個方法的優點是適用於這些帳戶存取的所有區域，而不需記得建立任何鎖定。 它的缺點則是具有區域刪除權限的任何帳戶，例如訂用帳戶擁有者仍可能不小心刪除重要區域。

可以同時使用這兩種方法 - 資源鎖定和自訂角色，做為 DNS 區域保護的深度防禦方法。

## <a name="next-steps"></a>後續步驟

* 如需使用 RBAC 的詳細資訊，請參閱[開始使用 Azure 入口網站中的存取管理](../role-based-access-control/overview.md)。
* 如需使用資源鎖定的詳細資訊，請參閱[使用 Azure Resource Manager 鎖定資源](../azure-resource-manager/resource-group-lock-resources.md)。

