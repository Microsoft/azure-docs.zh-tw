---
title: 針對雲端一致性的 Azure Resource Manager 範本 | Microsoft Docs
description: 開發針對雲端一致性的 Azure Resource Manager 範本。 建立新的或更新現有的 Azure Stack 範本。
services: azure-resource-manager
documentationcenter: na
author: marcvaneijk
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/05/2018
ms.author: mavane
ms.openlocfilehash: f1ff151c0b8d89910949d961b732c10901f19293
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38723370"
---
# <a name="develop-azure-resource-manager-templates-for-cloud-consistency"></a>開發針對雲端一致性的 Azure Resource Manager 範本

Azure 的主要優點在於一致性。 對某個位置的開發投資可重複用在另一個位置。 範本讓您的部署一致且可跨環境重複，包括全球 Azure、Azure 主權雲端和 Azure Stack。 但若要跨雲端重複使用範本，您需要考慮雲端特定的相依性，如本指南所述。

Microsoft 在許多位置提供符合企業需求的智慧型雲端服務，包括：

* Microsoft 受控之全球各區域資料中心中不斷成長的網路，支援全球 Azure 平台。
* 隔離式主權雲端，如 Azure 德國、Azure Government 和 Azure 中國 (21Vianet 所操作的 Azure)。 主權雲端提供一致的平台和大多數全球 Azure 客戶都能存取的相同絕佳功能。
* Azure Stack 是混合式雲端平台，可讓您從貴組織的資料中心提供 Azure 服務。 企業可以在自己的資料中心設定 Azure Stack，或從服務提供者取用 Azure 服務，在其設備 (有時稱為代管區域) 中執行 Azure Stack。

在所有這些雲端的核心，Azure Resource Manager 提供一個 API 讓各種不同的使用者介面都能與 Azure 平台通訊。 此 API 提供您強大的基礎結構即程式碼功能。 您可以使用 Azure Resource Manager 在 Azure 雲端平台上部署並設定任何類型的可用資源。 使用單一範本，您可以將完成的應用程式部署及設定成作業結束狀態。

![Azure 環境](./media/templates-cloud-consistency/environments.png)

全球 Azure、主權雲端、代管雲端及您資料中心的雲端一致性，可協助您從 Azure Resource Manager 獲益。 當您設定範本型資源部署和設定時，您可以跨這些雲端重複使用您的開發投資。

不過，雖然全球、主權、託管和混合式雲端能提供一致的服務，但並非所有的雲端都相同。 因此，您可以建立具有僅特定雲端所提供功能之相依性的範本。

本指南其餘部分會描述，針對 Azure Stack 規劃開發新的 Azure Resource Manager 範本或更新現有的範本時，要考量的區域。 您的檢查清單一般應包含下列項目：

* 確認目標部署位置可以使用您範本中的函式、端點、服務和其他資源。
* 將巢狀範本和設定成品儲存在可存取的位置，確保跨雲端的存取。
* 使用動態參考，而不是硬式編碼連結和項目。
* 請確定您使用的範本參數能在目標雲端中運作。
* 確認目標雲端可以使用資源特有的屬性。

如需 Azure Resource Manger 範本的介紹，請參閱[範本部署](resource-group-overview.md#template-deployment)。

## <a name="ensure-template-functions-work"></a>確定範本函式都能運作

Resource Manager 範本的基本語法是 JSON。 範本使用 JSON 的超集，擴展運算式和函式的語法。 範本語言處理器經常更新，以支援額外的範本函式。 如需可用範本函式的詳細說明，請參閱 [Azure Resource Manager 範本函式](resource-group-template-functions.md)。

Azure Resource Manager 中引入的新範本函式，不能立即提供主權雲端或 Azure Stack 使用。 若要成功部署範本，範本中參考的所有函式都必須能在目標雲端中使用。 

Azure Resource Manager 功能一律先引入全球 Azure。 您可以使用下列 PowerShell 指令碼確認 Azure Stack 是否可以使用新引入的範本函式： 

1. 建立 GitHub 存放庫的複製品：[https://github.com/marcvaneijk/arm-template-functions](https://github.com/marcvaneijk/arm-template-functions)。

1. 一旦您有存放庫的本機複製品，請使用 PowerShell 連線至目的地的 Azure Resource Manager。

1. 匯入 psm1 模組並執行 Test-AzureRmTemplateFunctions Cmdlet：

  ```powershell
  # Import the module
  Import-module <path to local clone>\AzureRmTemplateFunctions.psm1

  # Execute the Test-AzureRmTemplateFunctions cmdlet
  Test-AzureRmTemplateFunctions -path <path to local clone>
  ```

指令碼會部署多個最小化範本，每一個都只包含唯一的範本函式。 指令碼輸出會報告受支援但無法使用的範本函式。

## <a name="working-with-linked-artifacts"></a>使用連結的成品

範本可以包含已連結成品的參考，並包含連結至另一個範本的部署資源。 Resource Manager 會在執行階段擷取連結的範本 (也稱為巢狀範本)。 範本也可以包含虛擬機器 (VM) 延伸模組成品的參考。 這些成品是由 VM 延伸模組所擷取，此延伸模組於部署範本期間，在設定 VM 延伸模組的 VM 內執行。 

以下各節會描述在開發包含主部署範本外部成品的範本時，雲端一致性的考量。

### <a name="use-nested-templates-across-regions"></a>使用跨區域的巢狀範本

範本可以分解成可重複使用的小型範本，它們每一個都有特定用途，並可跨部署案例重複使用。 若要執行部署，您要指定稱為主要範本或主範本的單一範本。 它會指定要部署的資源，例如虛擬網路、VM 和 Web 應用程式。 主要範本也可以包含另一個範本的連結，這表示您可以巢狀化範本。 同樣地，巢狀範本可以包含其他範本的連結。 巢狀深度最多可有五層。

下列程式碼顯示 templateLink 參數如何參考巢狀範本：

```json
"resources": [
  {
     "apiVersion": "2017-05-10",
     "name": "linkedTemplate",
     "type": "Microsoft.Resources/deployments",
     "properties": {
       "mode": "incremental",
       "templateLink": {
          "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/vNet.json",
          "contentVersion":"1.0.0.0"
       }
     }
  }
]
```

Azure Resource Manager 在執行階段評估主要範本，然後擷取並評估每個巢狀範本。 擷取所有的巢狀範本之後，就會扁平化並開始進一步處理範本。

### <a name="make-linked-templates-accessible-across-clouds"></a>讓連結的範本能跨雲端存取

考慮在哪裡以及如何儲存您使用的所有範本。 Azure Resource Manager 會在執行階段擷取任何連結的範本，因此需要直接存取連結的範本。 常見做法是使用 GitHub 儲存巢狀範本。 GitHub 存放庫可以包含透過 URL 公開存取的檔案。 雖然這項技術在公用雲端和主權雲端運作良好，但 Azure Stack 環境可能位於公司網路或中斷連線的遠端位置，沒有任何連外的網際網路存取。 在這些情況下，Azure Resource Manager 會無法擷取巢狀範本。 

跨雲端部署的較佳做法是將您的連結範本儲存在可供目標雲端存取的位置。 理想的情況是，所有部署成品都是在持續整合/持續開發 (CI/CD) 管線中維護，並由此部署。 或者，您可以將巢狀範本儲存在 Blob 儲存體容器中，Azure Resource Manager 從容器擷取範本。 

因為每個雲端上的 Blob 儲存體使用不同的端點完整網域名稱 (FQDN)，以有兩個參數的連結範本位置設定範本。 參數可以在部署期間接受使用者輸入。 範本通常是由很多人撰寫與共用，因此最佳做法是使用這些參數的標準名稱。 命名慣例更有助於跨區域、雲端和作者重複使用範本。

在下列程式碼中，`_artifactsLocation` 用來指向單一位置，包含所有與部署相關的成品。 請注意，已提供預設值。 在部署期間，`_artifactsLocation` 如未指定任何輸入值，則會使用預設值。 `_artifactsLocationSasToken` 用為 `sasToken` 的輸入。 在 `_artifactsLocation` 不受保護的案例中 (例如，公用 GitHub 存放庫)，預設值應該是空字串。

```json
"parameters": {
  "_artifactsLocation": {
    "type": "string",
    "metadata": {
      "description": "The base URI where artifacts required by this template are located."
    },
    "defaultValue": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-custom-script-windows/"
  },
  "_artifactsLocationSasToken": {
    "type": "securestring",
    "metadata": {
      "description": "The sasToken required to access _artifactsLocation."
    },
    "defaultValue": ""
  }
}
```

在整個範本中，連結是結合基底 URI (來自 `_artifactsLocation` 參數) 以及成品相對路徑和 `_artifactsLocationSasToken` 所產生。 下列程式碼示範如何使用 uri 範本函式指定巢狀範本連結：

```json
"resources": [
  {
    "name": "shared",
    "type": "Microsoft.Resources/deployments",
    "apiVersion": "2015-01-01",
    "properties": {
      "mode": "Incremental",
      "templateLink": {
        "uri": "[uri(parameters('_artifactsLocation'), concat('nested/vnet.json', parameters('_artifactsLocationSasToken')))]",
        "contentVersion": "1.0.0.0"
      }
    }
  }
]
```

透過使用這種方法，就會使用 `_artifactsLocation` 參數的預設值。 如果需要從不同的位置擷取連結的範本，可在部署期間使用參數輸入覆寫預設值，不需要變更範本本身。

### <a name="use-artifactslocation-instead-of-hardcoding-links"></a>使用 _artifactsLocation 而非硬式編碼連結

除了用於巢狀範本，`_artifactsLocation` 參數中的 URL 還用為部署範本之所有相關成品的基底。 一些 VM 延伸模組包含儲存在範本外部的指令碼連結。 您不應該為這些延伸模組硬式編碼連結。 例如，自訂指令碼和 PowerShell DSC 延伸模組可連結至 GitHub 上的外部指令碼，如下所示： 

```json
"properties": {
  "publisher": "Microsoft.Compute",
  "type": "CustomScriptExtension",
  "typeHandlerVersion": "1.9",
  "autoUpgradeMinorVersion": true,
  "settings": {
    "fileUris": [
      "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
    ]
  }
}
```

將連結硬式編碼至指令碼有可能妨礙範本成功部署到其他位置。 在設定 VM 資源的期間，在 VM 內執行的 VM 代理程式會開始下載 VM 延伸模組中的所有連結的指令碼，並將這些指令碼儲存在 VM 的本機磁碟上。 此方法的功用如同＜使用跨區域的巢狀範本＞一節中所述的巢狀範本連結。

Resource Manager 會在執行階段擷取巢狀範本。 在 VM 延伸模組，由 VM 代理程式執行對任何外部成品的擷取。 除了成品擷取啟動器不同之外，範本定義中的解決方案是一樣的。 為 sasToken 輸入使用具有所有成品儲存所在之基底路徑預設值 (包括 VM 延伸模組指令碼) 的 _artifactsLocation 參數和 `_artifactsLocationSasToken` 參數。

```json
"parameters": {
  "_artifactsLocation": {
    "type": "string",
    "metadata": {
      "description": "The base URI where artifacts required by this template are located."
    },
    "defaultValue": "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/"
  },
  "_artifactsLocationSasToken": {
    "type": "securestring",
    "metadata": {
      "description": "The sasToken required to access _artifactsLocation."
    },
    "defaultValue": ""
  }
}
```

若要建構成品的絕對 URI，慣用方法是使用 uri 範本函式，而不是 concat 範本函式。 透過以 uri 範本函式取代 VM 延伸模組的指令碼硬式編碼連結，範本中的這項功能是針對雲端一致性所設定。

```json
"properties": {
  "publisher": "Microsoft.Compute",
  "type": "CustomScriptExtension",
  "typeHandlerVersion": "1.9",
  "autoUpgradeMinorVersion": true,
  "settings": {
    "fileUris": [
      "[uri(parameters('_artifactsLocation'), concat('scripts/configure-music-app.ps1', parameters('_artifactsLocationSasToken')))]"
    ]
  }
}
```

使用此方法，包括設定指令碼在內的所有部署成品，都會和範本本身儲存在同一位置。 您只需要為 _artifactsLocation 參數指定不同的基底 URL，即可變更所有連結的位置。

## <a name="factor-in-differing-regional-capabilities"></a>區別區域功能的因素

隨著敏捷式開發與 Azure 持續引入更新及新服務，提供的服務或更新[隨區域而不同](https://azure.microsoft.com/regions/services/)。 經過嚴格的內部測試之後，新服務或現有服務的更新通常會介紹給參與驗證程式的小眾客戶。 客戶成功通過驗證之後，就可在 Azure 區域的子集內使用服務或更新，再推廣到更多區域，推出至主權雲端，也有可能提供 Azure Stack 客戶使用。

了解 Azure 區域和雲端可能使用不同的服務，您就可以為您的範本制定一些主動式決策。 檢查雲端可用的資源提供者是不錯的起點。 資源提供者會告訴您一組可供 Azure 服務使用的資源和作業。

範本會部署與設定資源。 資源類型是由資源提供者提供。 例如，計算資源提供者 (Microsoft.Compute) 提供多種資源類型，例如 virtualMachines 和 availabilitySets。 每個資源提供者都會向 Azure Resource Manager 提供通用合約定義的 API，讓所有的資源提供者都能獲得一致統一的撰寫體驗。 不過，全球 Azure 提供的資源提供者可能無法在主權雲端或 Azure Stack 區域中使用。

![資源提供者](./media/templates-cloud-consistency/resource-providers.png) 

若要確認指定的雲端是否可以使用資源提供者，請在 Azure 命令列介面中 ([CLI](/cli/azure/install-azure-cli)) 執行下列程式碼：

```azurecli-interactive
az provider list --query "[].{Provider:namespace, Status:registrationState}" --out table
```

您也可以使用下列 PowerShell Cmdlet，查看可用的資源提供者：

```azurepowershell-interactive
Get-AzureRmResourceProvider -ListAvailable | Select-Object ProviderNamespace, RegistrationState
```

### <a name="verify-the-version-of-all-resource-types"></a>驗證所有資源類型的版本

所有資源類型都會有一組屬性，但每項資源也有自己專屬的特定屬性。 新功能和相關屬性有時會透過新的 API 版本新增至現有的資源類型。 範本中的資源有自己的 API 版本屬性 - `apiVersion`。 此版本控制可確保範本中現有的資源設定不受平台改變所影響。

引入至全球 Azure 現有資源類型的新 API 版本，可能無法立即提供所有區域、主權雲端或 Azure Stack 使用。 若要檢視可用之資源提供者、資源類型，和雲端 API 版本的清單，您可以使用 Azure 入口網站中的 [資源總管]。 在 [所有服務] 功能表中搜尋 [資源總管]。 在 [資源總管] 中展開 [提供者] 節點，傳回該雲端中所有可用的資源提供者、其資源類型，以及 API 版本。

若要使用 Azure CLI 列出指定雲端中所有資源類型的可用 API 版本，請執行下列指令碼：

```azurecli-interactive
az provider list --query "[].{namespace:namespace, resourceType:resourceType[]}"
```

您也可以使用下列 PowerShell Cmdlet：

```azurepowershell-interactive
Get-AzureRmResourceProvider | select-object ProviderNamespace -ExpandProperty ResourceTypes | ft ProviderNamespace, ResourceTypeName, ApiVersions
```

### <a name="refer-to-resource-locations-with-a-parameter"></a>參考具有參數的資源位置

範本一律部署到位於區域的資源群組。 除了部署本身之外，範本中的每個資源也會有您用來指定部署所在區域的位置屬性。 若要針對雲端一致性開發您的範本，您需要以動態方式參考資源位置，因為每個 Azure Stack 都會包含唯一的位置名稱。 通常資源會和資源群組部署在同一區域，但為支援跨區域應用程式可用性等案例，跨區域分配資源會很有用。

即使您可以在範本中指定資源屬性時硬式編碼區域名稱，但這種方法並不能保證可將範本部署到其他 Azure Stack 環境，因為那裡很可能沒有區域名稱。

請將輸入參數位置新增至具有預設值的範本，以容納不同的區域。 如果在部署期間未指定任何值，則會使用預設值。 

範本函式 `[resourceGroup()]` 會傳回包含下列索引鍵/值組的物件：

```json
{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}",
  "name": "{resourceGroupName}",
  "location": "{resourceGroupLocation}",
  "tags": {
  },
  "properties": {
    "provisioningState": "{status}"
  }
}
```

透過參考輸入參數 defaultValue 中物件的位置索引鍵，Azure Resource Manager 在執行階段以範本部署所在資源群組的位置名稱取代 `[resourceGroup().location]` 範本函式。

```json
"parameters": {
  "location": {
    "type": "string",
    "metadata": {
      "description": "Location the resources will be deployed to."
    },
    "defaultValue": "[resourceGroup().location]"
  }
},
"resources": [
  {
    "name": "storageaccount1",
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2015-06-15",
    "location": "[parameters('location')]",
    ...
```

使用此範本函式，您可以將範本部署至任何雲端，事先甚至不用知道區域名稱。 此外，範本中特定資源的位置可能和資源群組位置不同。 在此情況下，您可以使用該特定資源的其他輸入參數設定它，而同一範本中的其他資源仍然使用初始的位置輸入參數。

### <a name="track-versions-using-api-profiles"></a>使用 API 設定檔追蹤版本

追蹤 Azure Stack 中所有可用的資源提供者和相關的 API 版本，是非常有挑戰性的。 例如，在撰寫本文之際，Azure 中 **microsoft.compute/availabilitysets** 最新的 API 版本是 `2018-04-01`，但 Azure 和 Azure Stack 常見的可用 API 版本是 `2016-03-30`。 **microsoft.storage/storageaccounts** 在所有 Azure 和 Azure Stack 位置中共用的常見 API 版本是 `2016-01-01`，但 Azure 中最新的 API 版本是 `2018-02-01`。

因此，Resource Manager 將 API 設定檔的概念引入了範本。 沒有 API 設定檔，範本中每項資源都會使用 `apiVersion` 項目設定，此項目描述該特定資源的 API 版本。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "metadata": {
                "description": "Location the resources will be deployed to."
            },
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "variables": {},
    "resources": [
        {
            "name": "mystorageaccount",
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2016-01-01",
            "location": "[parameters('location')]",
            "properties": {
                "accountType": "Standard_LRS"
            }
        },
        {
            "name": "myavailabilityset",
            "type": "Microsoft.Compute/availabilitySets",
            "apiVersion": "2016-03-30",
            "location": "[parameters('location')]",
            "properties": {
                "platformFaultDomainCount": 2,
                "platformUpdateDomainCount": 2
            }
        }
    ],
    "outputs": {}
}
```

API 設定檔版本的作用為依據 Azure 和 Azure Stack 通用資源類型的單一 API 版本別名。 不是在範本中指定每個資源的 API 版本，您只在稱為 `apiProfile` 的新根項目中指定 API 設定檔版本，並省略個別資源的 `apiVersion` 項目。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "apiProfile": "2018–03-01-hybrid",
    "parameters": {
        "location": {
            "type": "string",
            "metadata": {
                "description": "Location the resources will be deployed to."
            },
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "variables": {},
    "resources": [
        {
            "name": "mystorageaccount",
            "type": "Microsoft.Storage/storageAccounts",
            "location": "[parameters('location')]",
            "properties": {
                "accountType": "Standard_LRS"
            }
        },
        {
            "name": "myavailabilityset",
            "type": "Microsoft.Compute/availabilitySets",
            "location": "[parameters('location')]",
            "properties": {
                "platformFaultDomainCount": 2,
                "platformUpdateDomainCount": 2
            }
        }
    ],
    "outputs": {}
}
```

API 設定檔確保可跨位置使用 API 版本，所以您不必手動驗證特定位置可否使用 apiVersions。 Azure Stack 操作員必須根據支援原則將解決方案保持在最新，以確保 Azure Stack 環境中有您 API 設定檔參考的 API 版本。 如果系統已過時六個月以上，會被視為不相容，必須更新環境。

API 設定檔不是範本中的必要項目。 即使您新增項目，它也只會用於未指定 `apiVersion` 的資源。 此項目允許漸進式變更，但不需要變更現有範本。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "apiProfile": "2018–03-01-hybrid",
    "parameters": {
        "location": {
            "type": "string",
            "metadata": {
                "description": "Location the resources will be deployed to."
            },
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "variables": {},
    "resources": [
        {
            "name": "mystorageaccount",
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2016-01-01",
            "location": "[parameters('location')]",
            "properties": {
                "accountType": "Standard_LRS"
            }
        },
        {
            "name": "myavailabilityset",
            "type": "Microsoft.Compute/availabilitySets",
            "location": "[parameters('location')]",
            "properties": {
                "platformFaultDomainCount": 2,
                "platformUpdateDomainCount": 2
            }
        }
    ],
    "outputs": {}
}
```

## <a name="check-endpoint-references"></a>檢查端點參考

資源可以參考平台上的其他服務。 例如，公用 IP 可以獲指派公用 DNS 名稱。 公用雲端、主權雲端和 Azure Stack 解決方案可有自己的不同端點命名空間。 在大部分情況下，資源在範本中只需要有前置詞作為輸入。 在執行階段，Azure Resource Manager 會將端點值附加在其後。 範本中必須明確指定某些端點值。 

> [!NOTE]
> 若要針對雲端一致性開發範本，請不要硬式編碼端點命名空間。

下列兩個範例是常見的端點命名空間，需要在建立資源時予以明確指定：

* 儲存體帳戶 (Blob、佇列、資料表和檔案)
* 資料庫和 Redis 快取的連接字串

在部署完成時，端點命名空間也可用為範本輸出的使用者資訊。 以下是常見的範例：

* 儲存體帳戶 (Blob、佇列、資料表和檔案)
* 連接字串 (MySql、SQLServer、SQLAzure、Custom、NotificationHub、ServiceBus、EventHub、ApiHub、DocDb、RedisCache、PostgreSQL)
* 流量管理員
* 公用 IP 位址的 domainNameLabel
* 雲端服務

在範本中，一般會避免硬式編碼的端點。 最佳做法是使用參考範本函式動態擷取端點。 例如，最常見的硬式編碼端點是儲存體帳戶的端點命名空間。 每個儲存體帳戶都有唯一的 FQDN，透過串連儲存體帳戶的名稱和端點命名空間所建構。 名為 mystorageaccount1 的 Blob 儲存體帳戶會導致不同的 FQDN，視雲端而定：

* 在全球 Azure 雲端上建立時，**mystorageaccount1.Blob.core.windows.net**。
* 在 Azure 中國雲端中建立時為 **mystorageaccount1.Blob.core.chinacloudapi.cn**。

下列參考範本函式會從儲存體資源提供者擷取端點命名空間：

```json
"diskUri":"[concat(reference(concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName')), '2015-06-15').primaryEndpoints.blob, 'container/myosdisk.vhd')]"
```

以 `reference` 範本函式取代儲存體帳戶端點的硬式編碼值，您可以使用相同的範本成功部署至不同的環境，卻不需要變更端點參考。

### <a name="refer-to-existing-resources-by-unique-id"></a>依唯一的識別碼參考現有的資源

您也可以參考同一雲端之相同租用戶中，相同或另一個訂用帳戶內相同或另一個資源群組中的現有資源。 若要擷取資源屬性，您必須使用資源本身的唯一識別碼。 `resourceId` 範本函式會擷取 SQL Server 等資源的唯一識別碼，如下列程式碼所示： 

```json
"outputs": {
  "resourceId":{
    "type": "string",
    "value": "[resourceId('otherResourceGroup', 'Microsoft.Sql/servers', parameters('serverName'))]"
  }
}
```

然後，您可以使用 `reference` 範本函式內的 `resourceId` 函式擷取資料庫的屬性。 傳回的物件包含保留完整端點值的 `fullyQualifiedDomainName` 屬性。 此值是在執行階段擷取，提供雲端環境專屬的端點命名空間。 若要定義連接字串，但不硬式編碼端點命名空間，您可以直接參考連接字串中的傳回物件屬性，如下所示：

```json
"[concat('Server=tcp:', reference(resourceId('sql', 'Microsoft.Sql/servers', parameters('test')), '2015-05-01-preview').fullyQualifiedDomainName, ',1433;Initial Catalog=', parameters('database'),';User ID=', parameters('username'), ';Password=', parameters('pass'), ';Encrypt=True;')]"
```

## <a name="consider-resource-properties"></a>考慮資源屬性

Azure Stack 環境內的特定資源有唯一屬性，您必須在範本中考慮這一點。

### <a name="ensure-vm-images-are-available"></a>確保 VM 映像可用

Azure 提供豐富的 VM 映像選項。 這些映像是由 Microsoft 和合作夥伴為部署所建立和準備。 這些映像構成平台的 VM 基礎。 不過，雲端一致的範本應該只參考可用的參數，特別是發行者、供應項目，以及可供全球 Azure、Azure 主權雲端或 Azure Stack 解決方案使用的 VM 映像 SKU。

若要擷取某個位置的可用 VM 映像清單，請執行下列 Azure CLI 命令：

```azurecli-interactive
az vm image list -all
```

您可以使用 Azure PowerShell Cmdlet [Get-azurermvmimagepublisher](/powershell/module/azurerm.compute/get-azurermvmimagepublisher) 擷取相同的清單，並使用 `-Location` 參數指定您想要的位置。 例如︰

```azurepowershell-interactive
Get-AzureRmVMImagePublisher -Location "West Europe" | Get-AzureRmVMImageOffer | Get-AzureRmVMImageSku | Get-AzureRMVMImage
```

此命令需要幾分鐘才會傳回全球 Azure 雲端之西歐區域的所有可用映像。

如果您讓 Azure Stack 可以使用這些 VM 映像，則會取用所有可用儲存體。 即使是最小的縮放單位，Azure Stack 為了讓您容納它，允許您選取想要新增至環境的映像。

下列程式碼範例顯示一致的方法，參考您 Azure Resource Manager 範本的發行者、供應項目和 SKU 參數：

```json
"storageProfile": {
    "imageReference": {
    "publisher": "MicrosoftWindowsServer",
    "offer": "WindowsServer",
    "sku": "2016-Datacenter",
    "version": "latest"
    }
}
```

### <a name="check-local-vm-sizes"></a>檢查本機 VM 大小

若要針對雲端一致性來開發範本，您需要確定所有目標環境都能提供您想要的 VM 大小。 VM 大小是一組效能特性和功能。 某些 VM 大小取決於 VM 執行所在的硬體。 例如，如果您想要部署 GPU 最佳化的 VM，則執行 Hypervisor 的硬體需要有硬體 GPU。

當 Microsoft 引入有特定硬體相依性的新 VM 大小時，此 VM 大小通常已先提供 Azure 雲端的一小部分區域使用。 然後提供其他區域和雲端使用。 若要確認您部署的每個雲端中是否存在此 VM 大小，您可以使用下列 Azure CLI 命令擷取可用的大小：

```azurecli-interactive
az vm list-sizes --location "West Europe"
```

如果是 Azure PowerShell，請使用：

```azurepowershell-interactive
Get-AzureRmVMSize -Location "West Europe"
```

如需可用服務的完整清單，請參閱[不同區域的產品](https://azure.microsoft.com/global-infrastructure/services/?cdn=disable)。

### <a name="check-use-of-azure-managed-disks-in-azure-stack"></a>檢查 Azure Stack 的 Azure 受控磁碟使用狀況

受控磁碟會處理 Azure 租用戶的儲存體。 您可以在部署 VM 時，使用受控磁碟以隱含的方式執行這些動作，不是明確建立儲存體帳戶以及指定虛擬硬碟 (VHD) 的 URI。 受控磁碟會將來自同一可用性集合之 VM 的所有磁碟放在不同的儲存體單位中，以增強可用性。 此外，現有的 VHD 從標準轉換成具有顯著較少停機時間的進階儲存體。

雖然受控磁碟位在 Azure Stack 藍圖上，但目前不受支援。 在獲得支援之前，您可以使用 VM 資源範本中的 `vhd` 項目明確指定 VHD，開發適用於 Azure Stack 的雲端一致範本，如下所示：

```json
"storageProfile": {
  "imageReference": {
    "publisher": "MicrosoftWindowsServer",
    "offer": "WindowsServer",
    "sku": "[parameters('windowsOSVersion')]",
    "version": "latest"
  },
  "osDisk": {
    "name": "osdisk",
    "vhd": {
      "uri": "[concat(reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName')), '2015-06-15').primaryEndpoints.blob, 'vhds/osdisk.vhd')]"
    },
    "caching": "ReadWrite",
    "createOption": "FromImage"
  }
}
```

相反地，若要在範本中指定受控磁碟設定，請從磁碟設定中移除 `vhd` 項目。

```json
"storageProfile": {
  "imageReference": {
    "publisher": "MicrosoftWindowsServer",
    "offer": "WindowsServer",
    "sku": "[parameters('windowsOSVersion')]",
    "version": "latest"
  },
  "osDisk": {
    "caching": "ReadWrite",
    "createOption": "FromImage"
  }
}
```

相同的變更也適用於[資料磁碟](../virtual-machines/windows/using-managed-disks-template-deployments.md)。

### <a name="verify-that-vm-extensions-are-available-in-azure-stack"></a>請確認 Azure Stack 中可以使用 VM 延伸模組

雲端一致性的另一項考量，是使用[虛擬機器延伸模組](../virtual-machines/windows/extensions-features.md)在 VM 內設定資源。 不是所有的 VM 延伸模組都可供 Azure Stack 使用。 範本可以指定 VM 延伸模組專用的資源，在範本內建立相依性和條件。

例如，如果您想要設定執行 Microsoft SQL Server 的 VM，則 VM 延伸模組可以將 SQL Server 設定為範本部署的一部分。 請考慮，如果部署範本包含的應用程式伺服器，也設定在執行 SQL Server 的 VM 上建立資料庫，會發生什麼情況。 除了也為應用程式伺服器使用 VM 延伸模組之外，您可以在成功傳回的 SQL Server VM 延伸模組資源上設定應用程式伺服器的相依性。 這個方法可確保已設定執行 SQL Server 的 VM 時，並能在應用程式伺服器接獲指示建立資料庫時供其使用。

範本的宣告式方法可讓您定義資源的結束狀態及其彼此之間的相依性，而平台負責相依性所需的邏輯。

#### <a name="check-that-vm-extensions-are-available"></a>檢查 VM 延伸模組是否可用

有許多類型的 VM 延伸模組。 針對雲端一致性開發範本時，請務必只使用範本所有目標區域都提供的延伸模組。

若要擷取可供特定區域使用的 VM 延伸模組清單 (在本範例中為 `myLocation`)，請執行下列 Azure CLI 命令：

```azurecli-interactive
az vm extension image list --location myLocation
```

您也可以執行 Azure PowerShell [Get-AzureRmVmImagePublisher](/powershell/module/azurerm.compute/get-azurermvmimagepublisher) Cmdlet，使用 `-Location` 指定虛擬機器映像的位置。 例如︰

```azurepowershell-interactive
Get-AzureRmVmImagePublisher -Location myLocation | Get-AzureRmVMExtensionImageType | Get-AzureRmVMExtensionImage | Select Type, Version
```

#### <a name="ensure-that-versions-are-available"></a>確保有可用的版本

因為 VM 延伸模組是第一方 Resource Manager 資源，所以它們有自己的 API 版本。 如下列程式碼所示，VM 延伸模組類型是 Microsoft.Compute 資源提供者中的巢狀資源。

```json
{
    "apiVersion": "2015-06-15",
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "name": "myExtension",
    "location": "[parameters('location')]",
    ...
```

VM 延伸模組資源的 API 版本必須存在於您打算使用範本鎖定目標的所有位置。 位置相依性的運作方式類似之前在＜驗證所有資源類型的版本＞一節中討論的資源提供者 API 版本可用性。

若要擷取 VM 延伸模組資源的可用 API 版本清單，請使用 [Get-AzureRmResourceProvider](/powershell/module/azurerm.resources/get-azurermresourceprovider) Cmdlet 和 **Microsoft.Compute** 資源提供者，如下所示：

```azurepowershell-interactive
Get-AzureRmResourceProvider -ProviderNamespace "Microsoft.Compute" | Select-Object -ExpandProperty ResourceTypes | Select ResourceTypeName, Locations, ApiVersions | where {$_.ResourceTypeName -eq "virtualMachines/extensions"}
```

您也可以使用虛擬機器擴展集中的 VM 延伸模組。 適用於相同的位置條件。 若要針對雲端一致性開發您的範本，請確定您計劃部署的所有位置都提供 API 版本。 若要擷取擴展集之 VM 延伸模組資源的 API 版本，請使用與前文相同的 Cmdlet，但如下指定虛擬機器擴展集資源類型：

```azurepowershell-interactive
Get-AzureRmResourceProvider -ProviderNamespace "Microsoft.Compute" | Select-Object -ExpandProperty ResourceTypes | Select ResourceTypeName, Locations, ApiVersions | where {$_.ResourceTypeName -eq "virtualMachineScaleSets/extensions"}
```

每個特定的延伸模組也已建立版本。 本版顯示在 VM 延伸模組的 `typeHandlerVersion` 屬性中。 請確定您範本之 VM 延伸模組 `typeHandlerVersion` 項目中指定的版本，可在您計劃部署範本的位置中使用。 例如，下列程式碼指定 1.7 版：

```json
{
    "name": "MyCustomScriptExtension",
    "type": "extensions",
    "apiVersion": "2016-03-30",
    "location": "[parameters('location')]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]"
    ],
    "properties": {
        "publisher": "Microsoft.Compute",
        "type": "CustomScriptExtension",
        "typeHandlerVersion": "1.7",
        ...   
```

若要擷取特定 VM 延伸模組的可用版本清單，請使用 [Get-AzureRmVMExtensionImage](/powershell/module/azurerm.compute/get-azurermvmextensionimage) Cmdlet。 下列範例會從 **myLocation** 擷取 PowerShell DSC (預期狀態設定) VM 延伸模組的可用版本：

```azurepowershell-interactive
Get-AzureRmVMExtensionImage -Location myLocation -PublisherName Microsoft.PowerShell -Type DSC | FT
```

若要取得可用發行者清單，請使用 [Get-AzureRmVmImagePublisher](/powershell/module/azurerm.compute/get-azurermvmimagepublisher) 命令。 若要要求類型，請使用 [Get-AzureRmVMExtensionImageType](/powershell/module/azurerm.compute/get-azurermvmextensionimagetype) 命令。

## <a name="tips-for-testing-and-automation"></a>測試和自動化祕訣

在撰寫範本時追蹤所有相關的設定、功能及限制是項挑戰。 常見的方法是先針對單一雲端開發與測試範本，再以其他位置為目標。 不過，愈早在撰寫程序中執行測試，您開發小組必須執行的疑難排解和程式碼重寫就愈少。 因為疑難排解位置相依性相當耗時而失敗的部署。 這就是為什麼我們建議盡早在撰寫週期中自動化測試。 最後，您會需要較少的開發時間和較少的資源，而您的雲端一致成品會變得更有價值。

下圖顯示使用整合式開發環境 (IDE) 之小組的開發程序典型範例。 在時間軸的不同階段，會執行不同的測試類型。 在這裡，兩名開發人員處理相同的解決方案，而此案例既適用於一位開發人員也適用於大型小組。 每位開發人員一般都會建立中央存放庫的本機複本，讓每個人都能在本機複本上工作，但不影響可能要使用相同檔案的其他人。

![工作流程](./media/templates-cloud-consistency/workflow.png) 

請考慮下列測試和自動化祕訣：

* 請務必使用測試工具。 例如，包含 IntelliSense 和其他功能的 Visual Studio Code 和 Visual Studio，可協助您驗證您的範本。
* 若要在本機 IDE 開發期間改善程式碼品質，請使用單元測試和整合測試來執行靜態程式碼分析。 
* 為在初始開發期間取得更好的體驗，單元測試和整合測試應該只在找到問題和繼續進行測試時提出警告。 如此一來，您可以找出問題來加以解決，並決定變更的優先處理順序，也稱為測試導向部署 (TDD)。
* 請注意，某些測試可以不必連線至 Azure Resource Manager 執行。 其他測試，例如測試範本部署，則需要 Resource Manager 執行某些無法離線執行的動作。
* 針對驗證 API 測試部署範本不等於實際部署。 而且，即使您部署來自本機檔案的範本，所有對範本中巢狀範本的參考都是由 Resource Manager 直接擷取，而 VM 延伸模組參考的成品是由在已部署 VM 內執行的 VM 代理程式所擷取。

## <a name="next-steps"></a>後續步驟

* [Azure Resource Manager 範本考量](../azure-stack/user/azure-stack-develop-templates.md)
* [Azure Resource Manager 範本最佳做法](resource-group-authoring-templates.md)
