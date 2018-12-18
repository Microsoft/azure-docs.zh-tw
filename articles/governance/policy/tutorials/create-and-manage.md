---
title: 使用 Azure 原則來建立和管理原則以強制執行組織合規性
description: 使用 Azure 原則來強制執行標準、符合法規合規性與稽核需求、控制成本、維護安全性與效能一致性，並且強制整個企業的設計原則。
services: azure-policy
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: tutorial
ms.service: azure-policy
ms.custom: mvc
manager: carmonm
ms.openlocfilehash: a7495ca1153fa6b84902423ee79c69f24316cb40
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46980952"
---
# <a name="create-and-manage-policies-to-enforce-compliance"></a>建立和管理原則來強制執行相容性

了解如何在 Azure 中建立和管理原則，對於保持符合公司標準和服務等級協定而言，相當重要。 在本教學課程中，您將了解如何使用 Azure 原則在整個組織執行與建立、指派及管理原則相關的某些常見工作，例如：

> [!div class="checklist"]
> - 指派原則來強制執行您在未來建立的資源條件
> - 建立及指派計畫定義，以追蹤多個資源的相容性
> - 解決不相容或拒絕的資源
> - 在整個組織中實作新的原則

如果您想要指派原則以識別現有資源的目前合規性狀態，快速入門文章會說明如何執行這項操作。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/)。

## <a name="assign-a-policy"></a>指派原則

使用 Azure 原則強制執行相容性的第一步是指派原則定義。 原則定義會定義在何種條件下強制執行原則，以及要發揮什麼作用。 在此範例中，指派稱為「需要 SQL Server 12.0 版」的內建原則定義，以強制執行所有 SQL Server 資料庫都必須是 v12.0 才能相容的條件。

1. 藉由按一下 [所有服務] 然後搜尋並選取 [原則]，在 Azure 入口網站中啟動 Azure 原則服務。

   ![搜尋原則](../media/create-and-manage/search-policy.png)

1. 選取 Azure 原則分頁左側的 [指派]。 指派是已指派在特定範圍內發生的原則。

   ![選取指派](../media/create-and-manage/select-assignments.png)

1. 從 [原則 - 指派] 分頁頂端選取 [指派原則]。

   ![指派原則定義](../media/create-and-manage/select-assign-policy.png)

1. 在 [指派原則] 分頁上，藉由按一下省略符號並選取管理群組或訂用帳戶來選取 [範圍]。 選取資源群組 (選用)。 範圍會決定在哪些資源或資源群組上強制執行原則指派。  然後，按一下位於 [範圍] 分頁底部的 [選取]。

   這個範例會使用 **Contoso 訂用帳戶**。 您的訂用帳戶不同。

1. 以**範圍**作為基礎，您可以將一些資源排除在外。  **排除項目**會在比**範圍**層級低的層級上啟動。 **排除項目**是選擇用項目，所以目前先保留空白。

1. 選取 [原則定義] 省略符號以開啟可用定義的清單。 您可以將原則定義 [類型] 篩選為 [內建]，以檢視所有項目並閱讀其描述。

1. 選取 [需要 SQL Server 12.0 版]。 如果您無法立即找到它，請在搜尋方塊中輸入 [需要 SQL Server]，然後按 ENTER 鍵或按一下 [搜尋] 方塊外面。 一旦您找到並選取原則定義後，請按一下 [可用的定義] 分頁底部的 [選取]。

   ![找出原則](../media/create-and-manage/select-available-definition.png)

1. [指派名稱] 會自動填入您選取的原則名稱，但您可加以變更。 在此範例中，請保留「需要 SQL Server 12.0 版」。 您也可以新增選擇性的 [描述]。 描述會提供有關此原則指派的詳細資料。  **指派者**會自動填入登入者。 這是選用欄位，所以可以輸入自訂值。

1. 不用勾選 [建立受控識別]。 但是當要指派的原則或計畫包含具有 [deployIfNotExists](../concepts/effects.md#deployifnotexists) 效果的原則時，「必須」勾選此項目。 因為本教學課程中所用的原則並沒有包含該效果，所以保留空白。 如需詳細資訊，請參閱[受控識別](../../../active-directory/managed-identities-azure-resources/overview.md)和[補救安全性的運作方式](../how-to/remediate-resources.md#how-remediation-security-works)。

1. 按一下 [指派]。

## <a name="implement-a-new-custom-policy"></a>實作新的自訂原則

既然您已指派內建原則定義，可以進一步利用 Azure 原則。 接下來，建立新的自訂原則，藉由確保在您的環境中建立的 VM 不是 G 系列來節省成本。 如此一來，每次您組織中的使用者嘗試建立 G 系列的 VM 時，要求就會遭到拒絕。

1. 選取 Azure 原則分頁左側 [製作] 下的 [定義]。

   ![定義正在製作中](../media/create-and-manage/definition-under-authoring.png)

1. 選取分頁頂端的 [+ 原則定義]。 這會開啟至 [原則定義] 分頁。

1. 輸入以下資訊：

   - 原則定義儲存所在的管理群組或訂用帳戶。 使用 [定義位置] 上的省略符號加以選取。

     > [!NOTE]
     > 如果您計劃要將此原則定義套用至多個訂用帳戶，則作為位置的管理群組必須包含您要對其指派原則的訂用帳戶。 計畫定義也是如此。

   - 原則定義的名稱 - 需要小於 G 系列的 VM SKU
   - 原則定義目的的描述 – 此原則定義會強制讓此範圍中建立的所有 VM，具有比 G 系列小的 SKU，以降低成本。
   - 從現有選項 (例如_計算_) 進行選擇，或為此原則定義建立新類別。
   - 複製下列 JSON 程式碼，然後針對您的需求進行更新：
      - 原則參數。
      - 原則規則/條件，在此情況下 – VM SKU 大小等於 G 系列
      - 原則效果，在此情況下 – **拒絕**。

    JSON 看起來應該會像下面這樣。 將修改過的程式碼貼到 Azure 入口網站中。

    ```json
    {
        "policyRule": {
            "if": {
                "allOf": [{
                        "field": "type",
                        "equals": "Microsoft.Compute/virtualMachines"
                    },
                    {
                        "field": "Microsoft.Compute/virtualMachines/sku.name",
                        "like": "Standard_G*"
                    }
                ]
            },
            "then": {
                "effect": "deny"
            }
        }
    }
    ```

    原則規則中欄位屬性的值必須是下列其中一項：Name、Type、Location、Tags 或別名。 可能的別名範例為 `"Microsoft.Compute/VirtualMachines/Size"`。

    若要檢視更多的 Azure 原則範例，請參閱 [Azure 原則範例](../samples/index.md)。

1. 選取 [ **儲存**]。

## <a name="create-a-policy-definition-with-rest-api"></a>使用 REST API 來建立原則定義

您可以使用「適用於原則定義的 REST API」來建立原則。 REST API 可讓您建立和刪除原則定義，以及取得現有定義的相關資訊。 若要建立原則定義，請使用下列範例：

```http-interactive
PUT https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.authorization/policydefinitions/{policyDefinitionName}?api-version={api-version}
```

納入如下範例的要求內文：

```json
{
    "properties": {
        "parameters": {
            "allowedLocations": {
                "type": "array",
                "metadata": {
                    "description": "The list of locations that can be specified when deploying resources",
                    "strongType": "location",
                    "displayName": "Allowed locations"
                }
            }
        },
        "displayName": "Allowed locations",
        "description": "This policy enables you to restrict the locations your organization can specify when deploying resources.",
        "policyRule": {
            "if": {
                "not": {
                    "field": "location",
                    "in": "[parameters('allowedLocations')]"
                }
            },
            "then": {
                "effect": "deny"
            }
        }
    }
}
```

## <a name="create-a-policy-definition-with-powershell"></a>使用 PowerShell 來建立原則定義

繼續進行 PowerShell 範例之前，請確定您已安裝最新版的 Azure PowerShell。 原則參數是於 3.6.0 版中加入。 如果您使用舊版本，範例會傳回找不到參數的錯誤。

您可以使用 `New-AzureRmPolicyDefinition` cmdlet 建立原則定義。

若要從檔案建立原則定義，請將路徑傳遞至檔案。 針對外部檔案，請使用下列範例：

```azurepowershell-interactive
$definition = New-AzureRmPolicyDefinition `
    -Name 'denyCoolTiering' `
    -DisplayName 'Deny cool access tiering for storage' `
    -Policy 'https://raw.githubusercontent.com/Azure/azure-policy-samples/master/samples/Storage/storage-account-access-tier/azurepolicy.rules.json'
```

針對本機檔案，請使用下列範例：

```azurepowershell-interactive
$definition = New-AzureRmPolicyDefinition `
    -Name 'denyCoolTiering' `
    -Description 'Deny cool access tiering for storage' `
    -Policy 'c:\policies\coolAccessTier.json'
```

若要建立具有內嵌規則的原則定義，請使用下列範例：

```azurepowershell-interactive
$definition = New-AzureRmPolicyDefinition -Name 'denyCoolTiering' -Description 'Deny cool access tiering for storage' -Policy '{
    "if": {
        "allOf": [{
                "field": "type",
                "equals": "Microsoft.Storage/storageAccounts"
            },
            {
                "field": "kind",
                "equals": "BlobStorage"
            },
            {
                "field": "Microsoft.Storage/storageAccounts/accessTier",
                "equals": "cool"
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}'
```

輸出會儲存在於原則指派期間使用的 `$definition` 物件中。 下列範例建立了包含參數的原則定義：

```azurepowershell-interactive
$policy = '{
    "if": {
        "allOf": [{
                "field": "type",
                "equals": "Microsoft.Storage/storageAccounts"
            },
            {
                "not": {
                    "field": "location",
                    "in": "[parameters(''allowedLocations'')]"
                }
            }
        ]
    },
    "then": {
        "effect": "Deny"
    }
}'

$parameters = '{
    "allowedLocations": {
        "type": "array",
        "metadata": {
            "description": "The list of locations that can be specified when deploying storage accounts.",
            "strongType": "location",
            "displayName": "Allowed locations"
        }
    }
}'

$definition = New-AzureRmPolicyDefinition -Name 'storageLocations' -Description 'Policy to specify locations for storage accounts.' -Policy $policy -Parameter $parameters
```

### <a name="view-policy-definitions-with-powershell"></a>使用 PowerShell 檢視原則定義

若要查看訂用帳戶中的所有原則定義，請使用下列命令：

```azurepowershell-interactive
Get-AzureRmPolicyDefinition
```

它會傳回所有可用的原則定義，包括內建原則。 每個原則都以下列格式傳回：

```output
Name               : e56962a6-4747-49cd-b67b-bf8b01975c4c
ResourceId         : /providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c
ResourceName       : e56962a6-4747-49cd-b67b-bf8b01975c4c
ResourceType       : Microsoft.Authorization/policyDefinitions
Properties         : @{displayName=Allowed locations; policyType=BuiltIn; description=This policy enables you to
                     restrict the locations your organization can specify when deploying resources. Use to enforce
                     your geo-compliance requirements.; parameters=; policyRule=}
PolicyDefinitionId : /providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c
```

## <a name="create-a-policy-definition-with-azure-cli"></a>使用 Azure CLI 來建立原則定義

您可以使用 Azure CLI 搭配原則定義命令來建立原則定義。 若要建立具有內嵌規則的原則定義，請使用下列範例：

```azurecli-interactive
az policy definition create --name 'denyCoolTiering' --description 'Deny cool access tiering for storage' --rules '{
    "if": {
        "allOf": [{
                "field": "type",
                "equals": "Microsoft.Storage/storageAccounts"
            },
            {
                "field": "kind",
                "equals": "BlobStorage"
            },
            {
                "field": "Microsoft.Storage/storageAccounts/accessTier",
                "equals": "cool"
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}'
```

### <a name="view-policy-definitions-with-azure-cli"></a>使用 Azure CLI 檢視原則定義

若要查看訂用帳戶中的所有原則定義，請使用下列命令：

```azurecli-interactive
az policy definition list
```

它會傳回所有可用的原則定義，包括內建原則。 每個原則都以下列格式傳回：

```json
{
    "description": "This policy enables you to restrict the locations your organization can specify when deploying resources. Use to enforce your geo-compliance requirements.",
    "displayName": "Allowed locations",
    "id": "/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c",
    "name": "e56962a6-4747-49cd-b67b-bf8b01975c4c",
    "policyRule": {
        "if": {
            "not": {
                "field": "location",
                "in": "[parameters('listOfAllowedLocations')]"
            }
        },
        "then": {
            "effect": "Deny"
        }
    },
    "policyType": "BuiltIn"
}
```

## <a name="create-and-assign-an-initiative-definition"></a>建立並指派計畫定義

使用計畫定義，您可以群組數個原則定義，以達成一個總體目標。 您建立計畫定義，以確保定義範圍內的資源保持與組成計畫定義的原則定義相容。 如需關於計畫定義的詳細資訊，請參閱 [Azure 原則概觀](../overview.md)。

### <a name="create-an-initiative-definition"></a>建立計畫定義

1. 選取 Azure 原則分頁左側 [製作] 下的 [定義]。

   ![選取定義](../media/create-and-manage/definition-under-authoring.png)

1. 選取分頁頂端的 [+ 計畫定義]，可開啟 [計畫定義] 分頁。

   ![計畫定義](../media/create-and-manage/initiative-definition.png)

1. 使用 [定義位置] 省略符號，選取要儲存定義的管理群組或訂用帳戶。 如果在上一頁中，已將範圍設為單一管理群組或訂用帳戶，則 [定義位置] 會自動填入。

1. 輸入計畫的 [名稱] 和 [說明]。

   此範例會確保資源都符合保障安全的原則定義。 將計畫命名為**保障安全**，並將描述設為：**已經建立計畫以處理與保護資源安全相關聯的所有原則定義**。

1. 針對 [類別]，從現有選項進行選擇，或建立新的類別。

1. 瀏覽 [可用定義] ([計畫定義] 分頁的右半部) 的清單，並選取您想要新增至此計畫的原則定義。 針對 [保障安全] 計畫，按一下原則定義資訊旁的 [+]，或按一下原則定義資料列，然後按一下詳細資料頁面中的 [+ 新增] 選項，以新增下列內建原則定義：

   - 需要 SQL Server 12.0 版
   - [Preview]: Monitor unprotected web applications in Security Center.
   - [Preview]: Monitor permissive network across in Security Center.
   - [Preview]: Monitor possible app Whitelisting in Security Center.
   - [Preview]: Monitor unencrypted VM Disks in Security Center.

   從清單中選取原則定義之後，系統會將該定義新增到 [原則和參數] 底下。

   ![計畫定義](../media/create-and-manage/initiative-definition-2.png)

1. 要新增至計畫的原則定義如果有參數，則會顯示在 [原則和參數] 區域的原則名稱下方。 「值」可以設定為 [設定值] (將此計畫的所有指派硬式編碼) 或 [使用計畫參數] (在每個計畫指派期間設定)。 如果選取 [設定值]，則 [值] 右側的下拉式清單會允許輸入或選取所需的值。 如果選取 [使用計畫參數]，則會顯示新的 [計畫參數] 區段，讓您定義將在計畫指派期間設定的參數。 此計畫參數的允許值可進一步限制可在計畫指派期間設定的項目。

   ![計畫定義參數](../media/create-and-manage/initiative-definition-3.png)

   > [!NOTE]
   > 就某些 `strongType` 參數而言，值清單是無法自動決定的。 在這些情況下，參數資料列的右側會出現省略符號。 按一下省略符號以開啟 [參數範圍 (&lt;參數名稱&gt;)] 分頁。 在此頁面上，請選取要用來提供值選項的訂用帳戶。 此參數範圍只會在建立計畫定義期間使用，且對於原則評估或指派的計畫範圍均無影響。

1. 按一下 [檔案] 。

### <a name="assign-an-initiative-definition"></a>指派計畫定義

1. 選取 Azure 原則分頁左側 [製作] 下的 [定義]。

1. 找出您先前建立的 [保障安全] 計畫定義並按一下它。 選取分頁頂端的 [指派] 以開啟至 [保障安全：指派計畫] 分頁。

   ![指派定義](../media/create-and-manage/assign-definition.png)

   或者，您也可以用滑鼠右鍵按一下選取的資料列，或用滑鼠左鍵按一下資料列結尾的省略符號，以顯示快顯功能表。  然後選取 [指派]。

   ![以滑鼠右鍵按一下資料列](../media/create-and-manage/select-right-click.png)

1. 輸入下列範例資訊，以填寫 [保障安全: 指派計畫] 分頁。 您可以使用自己的資訊。

   - 範圍：儲存計劃的管理群組或訂用帳戶會成為預設值。  您可以變更範圍，將計畫指派給儲存位置內的訂用帳戶或資源群組。
   - 排除項目：設定範圍內的任何資源，以防止計畫指派套用至這些資源。
   - 計畫定義和指派名稱：保障安全 (預先填入作為所指派計畫的名稱)。
   - 描述：此計畫指派適合強制執行此原則定義群組。
   - 指派者：會自動填入登入者。 這是選用欄位，所以可以輸入自訂值。

1. 不用勾選 [建立受控識別]。 但是當要指派的原則或計畫包含具有 [deployIfNotExists](../concepts/effects.md#deployifnotexists) 效果的原則時，「必須」勾選此項目。 因為本教學課程中所用的原則並沒有包含該效果，所以保留空白。 如需詳細資訊，請參閱[受控識別](../../../active-directory/managed-identities-azure-resources/overview.md)和[補救安全性的運作方式](../how-to/remediate-resources.md#how-remediation-security-works)。

1. 按一下 [指派]。

## <a name="check-initial-compliance"></a>檢查初始合規性

1. 選取 Azure 原則分頁左側的 [合規性]。

1. 找出 [保障安全] 計劃。 其 [合規性狀態]可能仍然是 [未啟動]。 按一下該計畫可取得指派程序的完整詳細資料。

   ![合規性 - 未啟動](../media/create-and-manage/compliance-status-not-started.png)

1. 當完成計畫指派之後，合規性分頁的 [合規性狀態] 會更新為 [符合規範]。

   ![合規性 - 符合規範](../media/create-and-manage/compliance-status-compliant.png)

1. 按一下計畫合規性分頁上的任何原則，即可開啟該原則的合規性詳細資料頁面。 此分頁提供資源層級上的合規性詳細資料。

## <a name="exempt-a-non-compliant-or-denied-resource-using-exclusion"></a>使用排除來免除不相容或拒絕的資源

遵循上述範例，在指派原則定義需要 SQL Server 12.0 版之後，使用 12.0 以外版本建立的 SQL Server 就會遭到拒絕。 在本節中，您可藉由在單一資源群組中建立排除項目，以逐步解決建立 SQL Server 的被拒絕嘗試。 排除項目會防止對該資源強制執行原則 (或計畫)。
在下列範例中，單一資源群組中允許任何 SQL 伺服器版本。 排除可以套用至訂用帳戶或資源群組，或者您可將排除範圍縮小為個別的資源。

由於指派的原則或計畫可在兩個位置中檢視，因此無法部署：

- 在作為部署目標的資源群組上：選取分頁左側的 [部署]，並按一下失敗部署的 [部署名稱]。 遭拒的資源將會以 [禁止] 狀態列出。 若要確認拒絕資源的原則或計畫和指派，請在 [部署概觀] 頁面上按一下 [失敗。如需詳細資訊，請按一下這裡 ->]。 視窗會在分頁右側開啟，並附上錯誤資訊。 在 [錯誤詳細資料] 下，會有相關原則物件的 GUID。

  ![部署遭原則指派拒絕](../media/create-and-manage/rg-deployment-denied.png)

- 在 Azure 原則分頁上：選取分頁左側的 [合規性]，並按一下 [需要 SQL Server 12.0 版] 原則。 在開啟的頁面上，您會看到 [拒絕] 計數增加。 在 [事件] 索引標籤下，您也會看到哪些人員所嘗試的部署遭到原則拒絕。

  ![指派原則的合規性概觀](../media/create-and-manage/compliance-overview.png)

在此範例中，Trent Baker 是 Contoso 的其中一個資深虛擬化專家，正在執行必要的工作。 我們必須授與他例外狀況，但我們在任何資源群組中不需要非 12.0 版 SQL 伺服器。 我們建立了新的資源群組 **SQLServers_Excluded**，且現在將授與此群組對此原則指派的例外狀況。

### <a name="update-assignment-with-exclusion"></a>使用排除項目更新指派

1. 選取 Azure 原則分頁左側 [製作] 下的 [指派]。

1. 瀏覽所有原則指派，並開啟 [需要 SQL Server 12.0 版] 指派。

1. 按一下省略符號並選取要排除的資源群組 (在此範例中為 SQLServers_Excluded) 以設定 [排除]。

   ![要求排除](../media/create-and-manage/request-exclusion.png)

   > [!NOTE]
   > 根據原則及其效果，也可將排除授與給指派範圍內資源群組中的特定資源。 由於此教學課程中使用 [拒絕] 效果，因此對已經存在的特定資源設定排除將沒有意義。

1. 依序按一下 [選取] 和 [儲存]。

在本節中，您可藉由在單一資源群組中建立排除，來解決拒絕嘗試建立禁止的 SQL 伺服器版本。

## <a name="clean-up-resources"></a>清除資源

如果您已完成使用本教學課程中的資源，請使用下列步驟來刪除前面建立的任何指派或定義：

1. 選取 Azure 原則分頁左側 [製作] 下的 [定義] (或 [指派]，如果您嘗試刪除指派)。

1. 搜尋您要移除的新計畫或原則定義 (或指派)。

1. 以滑鼠右鍵按一下資料列，或選取定義 (或指派) 結尾的省略符號，然後選取 [刪除定義] (或 [刪除指派])。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已成功完成下列項目：

> [!div class="checklist"]
> - 指派原則來強制執行您在未來建立的資源條件
> - 建立及指派計畫定義，以追蹤多個資源的相容性
> - 解決不相容或拒絕的資源
> - 在整個組織中實作新的原則

若要深入了解原則定義的結構，請閱讀這篇文章：

> [!div class="nextstepaction"]
> [Azure 原則定義結構](../concepts/definition-structure.md)