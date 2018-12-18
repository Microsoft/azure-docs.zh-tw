---
title: Azure Resource Manager 範本資源 | Microsoft Docs
description: 使用宣告式 JSON 語法說明 Azure Resource Manager 範本的資源區段。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
editor: tysonn
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/11/2018
ms.author: tomfitz
ms.openlocfilehash: 6723cf8cc18637c157b295361425357e1c47ec2e
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39007156"
---
# <a name="resources-section-of-azure-resource-manager-templates"></a>Azure Resource Manager 範本的 Resources 區段

在資源區段中，您會定義要部署或更新的資源。 此區段會變得複雜，因為您必須了解您要部署的類型才能提供正確的值。

## <a name="available-properties"></a>可用屬性

您會定義結構如下的資源：

```json
"resources": [
  {
      "condition": "<true-to-deploy-this-resource>",
      "apiVersion": "<api-version-of-resource>",
      "type": "<resource-provider-namespace/resource-type-name>",
      "name": "<name-of-the-resource>",
      "location": "<location-of-resource>",
      "tags": {
          "<tag-name1>": "<tag-value1>",
          "<tag-name2>": "<tag-value2>"
      },
      "comments": "<your-reference-notes>",
      "copy": {
          "name": "<name-of-copy-loop>",
          "count": <number-of-iterations>,
          "mode": "<serial-or-parallel>",
          "batchSize": <number-to-deploy-serially>
      },
      "dependsOn": [
          "<array-of-related-resource-names>"
      ],
      "properties": {
          "<settings-for-the-resource>",
          "copy": [
              {
                  "name": ,
                  "count": ,
                  "input": {}
              }
          ]
      },
      "sku": {
          "name": "<sku-name>",
          "tier": "<sku-tier>",
          "size": "<sku-size>",
          "family": "<sku-family>",
          "capacity": <sku-capacity>
      },
      "kind": "<type-of-resource>",
      "plan": {
          "name": "<plan-name>",
          "promotionCode": "<plan-promotion-code>",
          "publisher": "<plan-publisher>",
          "product": "<plan-product>",
          "version": "<plan-version>"
      },
      "resources": [
          "<array-of-child-resources>"
      ]
  }
]
```

| 元素名稱 | 必要 | 說明 |
|:--- |:--- |:--- |
| condition | 否 | 布林值，指出是否會在此部署期間佈建資源。 若為 `true`，則會在部署期間建立資源。 若為 `false`，則會略過此部署的資源。 |
| apiVersion |是 |要用來建立資源的 REST API 版本。 |
| type |是 |資源類型。 這個值是資源提供者的命名空間與資源類型的組合 (例如 **Microsoft.Storage/storageAccounts**)。 |
| name |是 |資源名稱。 此名稱必須遵循在 RFC3986 中定義的 URI 元件限制。 此外，將資源名稱公開到外部合作對象的 Azure 服務會驗證該名稱，確定並非嘗試詐騙其他身分識別。 |
| location |視情況而異 |所提供資源的支援地理位置。 您可以選取任何可用的位置，但通常選擇接近您的使用者的位置很合理。 通常，將彼此互動的資源放在相同區域也合乎常理。 大部分的資源類型都需要有位置，但某些類型 (例如角色指派) 不需要位置。 |
| tags |否 |與資源相關聯的標記。 套用標籤，既可以邏輯方式組織訂用帳戶中的資源。 |
| comments |否 |您在範本中記錄資源的註解 |
| copy |否 |如果需要多個執行個體，要建立的資源數目。 預設模式為平行。 如果您不想要同時部署所有或某些資源，請指定序列模式。 如需詳細資訊，請參閱[在 Azure Resource Manager 中建立資源的多個執行個體](resource-group-create-multiple.md)。 |
| dependsOn |否 |在部署這項資源之前必須部署的資源。 Resource Manager 會評估資源之間的相依性，並依正確的順序進行部署。 資源若不互相依賴，則會平行部署資源。 值可以是以逗號分隔的資源名稱或資源唯一識別碼清單。 只會列出此範本中已部署的資源。 此範本中未定義的資源必須已經存在。 避免加入不必要的相依性，因為可能會降低部署速度並產生循環相依性。 如需設定相依性的指引，請參閱[定義 Azure Resource Manager 範本中的相依性](resource-group-define-dependencies.md)。 |
| properties |否 |資源特定的組態設定。 屬性的值和您在 REST API 作業 (PUT 方法) 要求主體中提供來建立資源的值是一樣的。 您也可以指定複本陣列，以建立屬性的數個執行個體。 |
| sku | 否 | 某些資源允許以值定義要部署的 SKU。 例如，您可以指定儲存體帳戶的備援類型。 |
| kind | 否 | 某些資源允許以值定義您所部署的資源類型。 例如，您可以指定要建立的 Cosmos DB 類型。 |
| 計劃 | 否 | 某些資源允許以值定義要部署的計劃。 例如，您可以指定虛擬機器的 Marketplace 映像。 | 
| resources |否 |與正在定義的資源相依的下層資源。 只提供父資源的結構描述允許的資源類型。 子資源的完整類型包含父資源類型，例如 **Microsoft.Web/sites/extensions**。 沒有隱含父資源的相依性。 您必須明確定義該相依性。 |

## <a name="condition"></a>條件

當您必須在部署期間決定是否建立資源時，請使用 `condition` 項目。 此元素的值會解析為 true 或 false。 此值為 true 時，會建立資源。 此值為 false 時，則不會建立資源。 一般而言，當您想要建立新資源或使用現有資源時，就會使用此值。 例如，若要指定要部署新的儲存體帳戶或使用現有的儲存體帳戶，請使用：

```json
{
    "condition": "[equals(parameters('newOrExisting'),'new')]",
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[variables('storageAccountName')]",
    "apiVersion": "2017-06-01",
    "location": "[resourceGroup().location]",
    "sku": {
        "name": "[variables('storageAccountType')]"
    },
    "kind": "Storage",
    "properties": {}
}
```

如需使用 `condition` 項目的完整範例範本，請參閱[ 使用新的或現有的虛擬網路、儲存體和公用 IP 的 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-new-or-existing-conditions)。

## <a name="resource-specific-values"></a>資源特定的值

每個資源類型的 **apiVersion**、**type** 和 **properties** 元素都不同。 **sku**、**kind** 和 **plan** 元素適用於某些資源類型，但並非全部。 若要判斷這些屬性的值，請參閱[範本參考](/azure/templates/)。

## <a name="resource-names"></a>資源名稱

通常，您會使用 Resource Manager 中的三種資源名稱︰

* 必須是唯一的資源名稱。
* 不需要是唯一的資源名稱，但是您選擇提供的名稱可協助您識別資源。
* 可以是一般的資源名稱。

### <a name="unique-resource-names"></a>唯一的資源名稱

為任何有資料存取端點的資源類型，提供唯一的資源名稱。 一些需要唯一名稱的常見資源類型包括︰

* Azure 儲存體<sup>1</sup> 
* Azure App Service 中的 Web Apps 功能
* SQL Server
* Azure 金鑰保存庫
* Azure Redis 快取
* Azure Batch
* Azure 流量管理員
* Azure 搜尋服務
* Azure HDInsight

<sup>1</sup> 儲存體帳戶名稱也必須是小寫、長度不超過 24 個字元，而且沒有任何連字號。

設定名稱時，您可以手動建立唯一的名稱或使用 [uniqueString()](resource-group-template-functions-string.md#uniquestring) 函式來產生名稱。 您也可能想要將首碼或尾碼新增至 **uniqueString** 結果。 修改唯一的名稱可協助您更輕鬆地識別名稱的資源類型。 例如，您可以使用下列變數來產生儲存體帳戶的唯一名稱：

```json
"variables": {
    "storageAccountName": "[concat(uniqueString(resourceGroup().id),'storage')]"
}
```

### <a name="resource-names-for-identification"></a>用於識別的資源名稱
您可能想要將某些資源類型命名，但其名稱不必是唯一的。 針對這些資源類型，您可以提供能識別資源內容和資源類型的名稱。

```json
"parameters": {
    "vmName": { 
        "type": "string",
        "defaultValue": "demoLinuxVM",
        "metadata": {
            "description": "The name of the VM to create."
        }
    }
}
```

### <a name="generic-resource-names"></a>一般資源名稱
若為主要透過不同資源來存取的資源類型，您可以使用在範本中硬式編碼的一般名稱。 例如，您可以在 SQL Server 上設定防火牆規則的標準、一般名稱︰

```json
{
    "type": "firewallrules",
    "name": "AllowAllWindowsAzureIps",
    ...
}
```

## <a name="location"></a>位置
在部署範本時，您必須為每個資源提供一個位置。 不同的位置支援不同的資源類型。 若要查看特定資源類型可供您的訂用帳戶使用的位置清單，請使用 Azure PowerShell 或 Azure CLI。 

下列範例使用 PowerShell 來取得 `Microsoft.Web\sites` 資源類型的位置︰

```powershell
((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes | Where-Object ResourceTypeName -eq sites).Locations
```

下列範例使用 Azure CLI 來取得 `Microsoft.Web\sites` 資源類型的位置：

```azurecli
az provider show -n Microsoft.Web --query "resourceTypes[?resourceType=='sites'].locations"
```

決定支援資源的位置之後，您可在範本中設定該位置。 設定此值最簡單的方式是在支援資源類型的位置建立一個資源群組，而且將每個位置設定為 `[resourceGroup().location]`。 您可以將範本重新部署到不同位置的資源群組，而且無需變更範本中的任何值或參數。 

下列範例顯示部署到與資源群組相同位置的儲存體帳戶：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "variables": {
      "storageName": "[concat('storage', uniqueString(resourceGroup().id))]"
    },
    "resources": [
    {
      "apiVersion": "2016-01-01",
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('storageName')]",
      "location": "[resourceGroup().location]",
      "tags": {
        "Dept": "Finance",
        "Environment": "Production"
      },
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": { }
    }
    ]
}
```

如果您需要在範本中採硬式編碼來指定位置，請提供其中一個支援區域的名稱。 下列範例顯示一律會部署到美國中北部的儲存體帳戶：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
    {
      "apiVersion": "2016-01-01",
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[concat('storageloc', uniqueString(resourceGroup().id))]",
      "location": "North Central US",
      "tags": {
        "Dept": "Finance",
        "Environment": "Production"
      },
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": { }
    }
    ]
}
```

## <a name="tags"></a>標記
[!INCLUDE [resource-manager-governance-tags](../../includes/resource-manager-governance-tags.md)]

### <a name="add-tags-to-your-template"></a>將標籤新增到您的範本

[!INCLUDE [resource-manager-tags-in-templates](../../includes/resource-manager-tags-in-templates.md)]

## <a name="child-resources"></a>子資源

在某些資源類型內，您也可以定義子資源陣列。 子資源是只存在於另一個資源內容中的資源。 例如，若沒有 SQL Server，SQL 資料庫就無法存在，所以資料庫是伺服器的子系。 您可以在伺服器的定義中定義資料庫。

```json
{
  "name": "exampleserver",
  "type": "Microsoft.Sql/servers",
  "apiVersion": "2014-04-01",
  ...
  "resources": [
    {
      "name": "exampledatabase",
      "type": "databases",
      "apiVersion": "2014-04-01",
      ...
    }
  ]
}
```

形成巢狀後，類型會設為 `databases`，但是其完整資源類型為 `Microsoft.Sql/servers/databases`。 您未提供 `Microsoft.Sql/servers/`，因為它被認為是來自父資源類型。 子資源名稱會設為 `exampledatabase`，但是完整名稱包含父系名稱。 您未提供 `exampleserver`，因為它被認為是來自父資源。

子資源類型的格式如下︰`{resource-provider-namespace}/{parent-resource-type}/{child-resource-type}`

子資源名稱的格式如下︰`{parent-resource-name}/{child-resource-name}`

但是，您不必在伺服器內定義資料庫。 您可以定義最上層的子資源。 如果父資源並未部署在相同範本中，或者想要使用 `copy` 來建立多個子資源，您可以使用此方法。 使用這個方法，您必須提供完整資源類型，並將父資源名稱納入子資源名稱中。

```json
{
  "name": "exampleserver",
  "type": "Microsoft.Sql/servers",
  "apiVersion": "2014-04-01",
  "resources": [ 
  ],
  ...
},
{
  "name": "exampleserver/exampledatabase",
  "type": "Microsoft.Sql/servers/databases",
  "apiVersion": "2014-04-01",
  ...
}
```

當建構資源的完整參考時，要從類型和名稱合併區段的順序並非只是將兩個串連。 相反地，在命名空間之後，使用從最特定到最不特定的一連串*類型/名稱*組：

```json
{resource-provider-namespace}/{parent-resource-type}/{parent-resource-name}[/{child-resource-type}/{child-resource-name}]*
```

例如︰

`Microsoft.Compute/virtualMachines/myVM/extensions/myExt` 為正確 `Microsoft.Compute/virtualMachines/extensions/myVM/myExt` 為不正確

## <a name="recommendations"></a>建議
當您使用資源時，下列資訊可能會很有幫助︰

* 針對範本中的每個資源指定 **comments**，協助其他參與者了解資源的用途：
   
   ```json
   "resources": [
     {
         "name": "[variables('storageAccountName')]",
         "type": "Microsoft.Storage/storageAccounts",
         "apiVersion": "2016-01-01",
         "location": "[resourceGroup().location]",
         "comments": "This storage account is used to store the VM disks.",
         ...
     }
   ]
   ```

* 如果您使用範本中的公用端點 (例如 Azure Blob 儲存體公用端點)，請勿將命名空間硬式編碼。 使用 **reference** 函式，動態擷取命名空間。 您可以使用此方法可將範本部署到不同的公用命名空間環境中，而不需要將範本中的端點手動變更。 將 API 版本設定成與您在範本中用於儲存體帳戶相同的版本：
   
   ```json
   "osDisk": {
       "name": "osdisk",
       "vhd": {
           "uri": "[concat(reference(concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName')), '2016-01-01').primaryEndpoints.blob, variables('vmStorageAccountContainerName'), '/',variables('OSDiskName'),'.vhd')]"
       }
   }
   ```
   
   如果將該儲存體帳戶部署在您要建立的相同範本中，當您參考資源時，即不需要指定提供者命名空間。 下列範例顯示簡化的語法：
   
   ```json
   "osDisk": {
       "name": "osdisk",
       "vhd": {
           "uri": "[concat(reference(variables('storageAccountName'), '2016-01-01').primaryEndpoints.blob, variables('vmStorageAccountContainerName'), '/',variables('OSDiskName'),'.vhd')]"
       }
   }
   ```
   
   如果您範本中其他的值是使用公用命名空間進行設定，請變更這些值以反映相同的 **reference** 函式。 例如，您可以設定虛擬機器診斷設定檔的 **storageUri** 屬性：
   
   ```json
   "diagnosticsProfile": {
       "bootDiagnostics": {
           "enabled": "true",
           "storageUri": "[reference(concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName')), '2016-01-01').primaryEndpoints.blob]"
       }
   }
   ```
   
   您也可以參考不同資源群組中現有的儲存體帳戶：

   ```json
   "osDisk": {
       "name": "osdisk", 
       "vhd": {
           "uri":"[concat(reference(resourceId(parameters('existingResourceGroup'), 'Microsoft.Storage/storageAccounts/', parameters('existingStorageAccountName')), '2016-01-01').primaryEndpoints.blob,  variables('vmStorageAccountContainerName'), '/', variables('OSDiskName'),'.vhd')]"
       }
   }
   ```

* 只有在應用程式要求時，才將公用 IP 位址指派給虛擬機器。 若要連線至虛擬機器 (VM) 進行偵錯，或要進行管理或系統管理，請使用輸入 NAT 規則、虛擬網路閘道或 Jumpbox。
   
     如需有關如何連線至虛擬機器的詳細資訊，請參閱︰
   
   * [在 Azure 中執行多層式架構的 VM](../guidance/guidance-compute-n-tier-vm.md)
   * [在 Azure Resource Manager 中設定 VM 的 WinRM 存取](../virtual-machines/windows/winrm.md)
   * [允許使用 Azure 入口網站從外部存取您的 VM](../virtual-machines/windows/nsg-quickstart-portal.md)
   * [允許使用 PowerShell 從外部存取您的 VM](../virtual-machines/windows/nsg-quickstart-powershell.md)
   * [允許使用 Azure CLI 從外部存取您的 Linux VM](../virtual-machines/virtual-machines-linux-nsg-quickstart.md)
* 公用 IP 位址的 **domainNameLabel** 屬性必須是唯一的。 **domainNameLabel** 值的長度必須介於 3 到 63 個字元之間，還要遵循這個規則運算式指定的規則：`^[a-z][a-z0-9-]{1,61}[a-z0-9]$`。 **uniqueString** 函式會產生長度為 13 個字元的字串，因此 **dnsPrefixString** 參數會限制為 50 個字元：

   ```json
   "parameters": {
       "dnsPrefixString": {
           "type": "string",
           "maxLength": 50,
           "metadata": {
               "description": "The DNS label for the public IP address. It must be lowercase. It should match the following regular expression, or it will raise an error: ^[a-z][a-z0-9-]{1,61}[a-z0-9]$"
           }
       }
   },
   "variables": {
       "dnsPrefix": "[concat(parameters('dnsPrefixString'),uniquestring(resourceGroup().id))]"
   }
   ```

* 將密碼新增至自訂指令碼擴充功能時，使用 **protectedSettings** 屬性中的 **commandToExecute** 屬性。
   
   ```json
   "properties": {
       "publisher": "Microsoft.Azure.Extensions",
       "type": "CustomScript",
       "typeHandlerVersion": "2.0",
       "autoUpgradeMinorVersion": true,
       "settings": {
           "fileUris": [
               "[concat(variables('template').assets, '/lamp-app/install_lamp.sh')]"
           ]
       },
       "protectedSettings": {
           "commandToExecute": "[concat('sh install_lamp.sh ', parameters('mySqlPassword'))]"
       }
   }
   ```
   
   > [!NOTE]
   > 為了確保作為參數傳遞至 VM 和擴充功能的祕密會經過加密，請使用相關擴充功能的 **protectedSettings** 屬性。
   > 
   > 


## <a name="next-steps"></a>後續步驟
* 若要檢視許多不同類型解決方案的完整範本，請參閱 [Azure 快速入門範本](https://azure.microsoft.com/documentation/templates/)。
* 如需您可以在範本內使用哪些函式的詳細資料，請參閱 [Azure Resource Manager 範本函式](resource-group-template-functions.md)。
* 若要在部署期間使用多個範本，請參閱[透過 Azure Resource Manager 使用連結的範本](resource-group-linked-templates.md)。
* 您可能需要使用不同資源群組內的資源。 這案例常見於使用多個資源群組之間所共用的儲存體帳戶或虛擬網路時。 如需詳細資訊，請參閱 [resourceId 函式](resource-group-template-functions-resource.md#resourceid)。
* 如需有關資源名稱限制的詳細資訊，請參閱 [Recommended naming conventions for Azure resources (Azure 資源的建議命名慣例)](../guidance/guidance-naming-conventions.md)。