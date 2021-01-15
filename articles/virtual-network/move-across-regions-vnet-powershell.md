---
title: 使用 Azure PowerShell 將 Azure 虛擬網路移至另一個 Azure 區域
description: 使用 Resource Manager 範本和 Azure PowerShell，將 Azure 虛擬網路從一個 Azure 區域移至另一個區域。
author: asudbring
ms.service: virtual-network
ms.topic: how-to
ms.date: 08/26/2019
ms.author: allensu
ms.openlocfilehash: bc504034f8d4565dd365b8d92dc2b2e6eadc1dae
ms.sourcegitcommit: d59abc5bfad604909a107d05c5dc1b9a193214a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98223324"
---
# <a name="move-an-azure-virtual-network-to-another-region-by-using-azure-powershell"></a>使用 Azure PowerShell 將 Azure 虛擬網路移至另一個區域

有各種不同的案例可將現有的 Azure 虛擬網路從一個區域移至另一個區域。 例如，您可能想要使用與現有虛擬網路相同的測試和可用性設定來建立虛擬網路。 或者，您可能想要將生產虛擬網路移至另一個區域，作為嚴重損壞修復計畫的一部分。

您可以使用 Azure Resource Manager 範本來完成將虛擬網路移至另一個區域的工作。 做法是將虛擬網路匯出至範本、修改參數以符合目的地區域，然後將範本部署到新的區域。 如需 Resource Manager 範本的詳細資訊，請參閱 [將資源群組匯出至範本](../azure-resource-manager/management/manage-resource-groups-powershell.md#export-resource-groups-to-templates)。


## <a name="prerequisites"></a>先決條件

- 請確定您的虛擬網路位於您想要移動的 Azure 區域中。

- 若要匯出虛擬網路並部署範本，以在另一個區域中建立虛擬網路，您必須擁有「網路參與者」角色或更高的角色。

- 虛擬網路對等互連將不會重新建立，如果它們仍然存在於範本中，它們將會失敗。 匯出範本之前，您必須移除任何虛擬網路對等。 然後，您可以在虛擬網路移動之後重新建立它們。
    
- 識別來源網路配置，以及您目前使用的所有資源。 此版面配置包含但不限於負載平衡器、網路安全性群組 (Nsg) 和公用 Ip。

- 確認您的 Azure 訂用帳戶可讓您在目的地區域中建立虛擬網路。 若要啟用所需的配額，請聯絡支援人員。

- 請確定您的訂用帳戶有足夠的資源，可支援新增此程式的虛擬網路。 如需詳細資訊，請參閱 [Azure 訂用帳戶和服務限制、配額和條件約束](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits)。


## <a name="prepare-for-the-move"></a>準備移動
在本節中，您會使用 Resource Manager 範本來準備要進行移動的虛擬網路。 然後，您可以使用 Azure PowerShell 命令將虛擬網路移至目的地區域。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

若要匯出虛擬網路，並使用 PowerShell 部署目標虛擬網路，請執行下列動作：

1. 使用 [disconnect-azaccount](/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0) 命令登入您的 Azure 訂用帳戶，然後遵循畫面上的指示：
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```

1. 取得您想要移至目的地區域之虛擬網路的資源識別碼，然後使用 [new-azvirtualnetwork](/powershell/module/az.network/get-azvirtualnetwork?view=azps-2.6.0)將它放置在變數中：

    ```azurepowershell-interactive
    $sourceVNETID = (Get-AzVirtualNetwork -Name <source-virtual-network-name> -ResourceGroupName <source-resource-group-name>).Id
    ```

1. 將來源虛擬網路匯出到您執行命令匯出的目錄中的 json 檔案 [->new-azresourcegroup](/powershell/module/az.resources/export-azresourcegroup?view=azps-2.6.0)：
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceVNETID -IncludeParameterDefaultValue
   ```

1. 下載的檔案與匯出資源的資源群組具有相同的名稱。 找出您使用命令匯出的 *\<resource-group-name> json* 檔案，然後在編輯器中開啟該檔案：
   
   ```azurepowershell
   notepad <source-resource-group-name>.json
   ```

1. 若要編輯虛擬網路名稱的參數，請將來源虛擬網路名稱的 **defaultValue** 屬性變更為您目標虛擬網路的名稱。 請務必以引號括住名稱。
    
    ```json
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentmyResourceGroupVNET.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "virtualNetworks_myVNET1_name": {
        "defaultValue": "<target-virtual-network-name>",
        "type": "String"
        }
    ```

1. 若要編輯將移動虛擬網路的目的地區域，請變更 [資源] 底下的 [ **位置** ] 屬性：

    ```json
    "resources": [
                {
                    "type": "Microsoft.Network/virtualNetworks",
                    "apiVersion": "2019-06-01",
                    "name": "[parameters('virtualNetworks_myVNET1_name')]",
                    "location": "<target-region>",
                    "properties": {
                        "provisioningState": "Succeeded",
                        "resourceGuid": "6e2652be-35ac-4e68-8c70-621b9ec87dcb",
                        "addressSpace": {
                            "addressPrefixes": [
                                "10.0.0.0/16"
                            ]
                        },

    ```
  
1. 若要取得區域位置代碼，您可執行下列命令來使用 Azure PowerShell Cmdlet [Get-AzLocation](/powershell/module/az.resources/get-azlocation?view=azps-1.8.0)：

    ```azurepowershell-interactive

    Get-AzLocation | format-table
    ```

1.  (選擇性) 您也可以根據您的需求變更 *\<resource-group-name> json* 檔案中的其他參數：

    * **位址空間**：儲存檔案之前，您可以修改 **資源**  >  **addressSpace** 區段並變更 **addressPrefixes** 屬性，以改變虛擬網路的位址空間：

        ```json
                "resources": [
                    {
                    "type": "Microsoft.Network/virtualNetworks",
                    "apiVersion": "2019-06-01",
                    "name": "[parameters('virtualNetworks_myVNET1_name')]",
                    "location": "<target-region",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "resourceGuid": "6e2652be-35ac-4e68-8c70-621b9ec87dcb",
                    "addressSpace": {
                        "addressPrefixes": [
                        "10.0.0.0/16"
                        ]
                    },
        ```

    * **子網**：您可以藉由變更檔案的 **子網** 區段來變更或新增至子網名稱和子網位址空間。 您可以藉由變更 [ **名稱** ] 屬性來變更子網的名稱。 您可以藉由變更 **addressPrefix** 屬性來變更子網位址空間：

        ```json
                "subnets": [
                    {
                    "name": "subnet-1",
                    "etag": "W/\"d9f6e6d6-2c15-4f7c-b01f-bed40f748dea\"",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "addressPrefix": "10.0.0.0/24",
                    "delegations": [],
                    "privateEndpointNetworkPolicies": "Enabled",
                    "privateLinkServiceNetworkPolicies": "Enabled"
                    }
                    },
                    {
                    "name": "GatewaySubnet",
                    "etag": "W/\"d9f6e6d6-2c15-4f7c-b01f-bed40f748dea\"",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "addressPrefix": "10.0.1.0/29",
                    "serviceEndpoints": [],
                    "delegations": [],
                    "privateEndpointNetworkPolicies": "Enabled",
                    "privateLinkServiceNetworkPolicies": "Enabled"
                    }
                    }

                ]
        ```

        若要變更位址首碼，請在下列兩個位置中編輯檔案：在上一節的程式碼中，以及在下列程式碼的 **type** 區段中編輯檔案。 變更下列程式碼中的 **addressPrefix** 屬性，以符合上一節程式碼中的 **addressPrefix** 屬性。

        ```json
         "type": "Microsoft.Network/virtualNetworks/subnets",
           "apiVersion": "2019-06-01",
           "name": "[concat(parameters('virtualNetworks_myVNET1_name'), '/GatewaySubnet')]",
              "dependsOn": [
                 "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworks_myVNET1_name'))]"
                   ],
              "properties": {
                 "provisioningState": "Succeeded",
                 "addressPrefix": "10.0.1.0/29",
                 "serviceEndpoints": [],
                 "delegations": [],
                 "privateEndpointNetworkPolicies": "Enabled",
                 "privateLinkServiceNetworkPolicies": "Enabled"
                  }
                 },
                  {
                  "type": "Microsoft.Network/virtualNetworks/subnets",
                  "apiVersion": "2019-06-01",
                  "name": "[concat(parameters('virtualNetworks_myVNET1_name'), '/subnet-1')]",
                     "dependsOn": [
                        "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworks_myVNET1_name'))]"
                          ],
                     "properties": {
                        "provisioningState": "Succeeded",
                        "addressPrefix": "10.0.0.0/24",
                        "delegations": [],
                        "privateEndpointNetworkPolicies": "Enabled",
                        "privateLinkServiceNetworkPolicies": "Enabled"
                         }
                  }
         ]
        ```

1. 儲存 *\<resource-group-name>.json* 檔案。

1. 使用 [>new-azresourcegroup](/powershell/module/az.resources/new-azresourcegroup?view=azps-2.6.0)建立目標虛擬網路所要部署的目的地區域中的資源群組：
    
    ```azurepowershell-interactive
    New-AzResourceGroup -Name <target-resource-group-name> -location <target-region>
    ```
    
1. 使用 [>new-azresourcegroupdeployment](/powershell/module/az.resources/new-azresourcegroupdeployment?view=azps-2.6.0)將已編輯的 *\<resource-group-name> json* 檔案部署到您在上一個步驟中建立的資源群組：

    ```azurepowershell-interactive

    New-AzResourceGroupDeployment -ResourceGroupName <target-resource-group-name> -TemplateFile <source-resource-group-name>.json
    ```

1. 若要確認已在目的地區域中建立資源，請使用 [>new-azresourcegroup](/powershell/module/az.resources/get-azresourcegroup?view=azps-2.6.0) 和 [new-azvirtualnetwork](/powershell/module/az.network/get-azvirtualnetwork?view=azps-2.6.0)：
    
    ```azurepowershell-interactive

    Get-AzResourceGroup -Name <target-resource-group-name>
    ```

    ```azurepowershell-interactive

    Get-AzVirtualNetwork -Name <target-virtual-network-name> -ResourceGroupName <target-resource-group-name>
    ```

## <a name="delete-the-virtual-network-or-resource-group"></a>刪除虛擬網路或資源群組 

部署虛擬網路之後，若要在目的地區域中重新開機或捨棄虛擬網路，請刪除您在目的地區域中建立的資源群組，將會刪除已移動的虛擬網路。 

若要移除資源群組，請使用 [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup?view=azps-2.6.0)：

```azurepowershell-interactive

Remove-AzResourceGroup -Name <target-resource-group-name>
```

## <a name="clean-up"></a>清除

若要認可您的變更並完成虛擬網路移動，請執行下列其中一項動作：

* 使用 [移除->new-azresourcegroup](/powershell/module/az.resources/remove-azresourcegroup?view=azps-2.6.0)來刪除資源群組：

    ```azurepowershell-interactive

    Remove-AzResourceGroup -Name <source-resource-group-name>
    ```

* 使用 [移除-new-azvirtualnetwork](/powershell/module/az.network/remove-azvirtualnetwork?view=azps-2.6.0)來刪除來源虛擬網路：  
    ``` azurepowershell-interactive

    Remove-AzVirtualNetwork -Name <source-virtual-network-name> -ResourceGroupName <source-resource-group-name>
    ```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已使用 PowerShell 將虛擬網路從一個區域移至另一個區域，然後清除不需要的來源資源。 若要深入瞭解如何在 Azure 中的區域與災難復原之間移動資源，請參閱：

- [將資源移到新的資源群組或訂用帳戶](../azure-resource-manager/management/move-resource-group-and-subscription.md) \(部分機器翻譯\)
- [將 Azure 虛擬機器移至另一個區域](../site-recovery/azure-to-azure-tutorial-migrate.md)