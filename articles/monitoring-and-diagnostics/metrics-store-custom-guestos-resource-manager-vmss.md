---
title: 使用 Windows 虛擬機器擴展集的 Resource Manager 範本將客體作業系統計量傳送至 Azure 監視器計量存放區
description: 使用 Windows 虛擬機器擴展集的 Resource Manager 範本將客體作業系統計量傳送至 Azure 監視器計量存放區
author: anirudhcavale
services: azure-monitor
ms.service: azure-monitor
ms.topic: howto
ms.date: 09/24/2018
ms.author: ancav
ms.component: metrics
ms.openlocfilehash: b9808233e08e545c31e171afe104173dccc6abed
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47434920"
---
# <a name="send-guest-os-metrics-to-the-azure-monitor-metric-store-using-a-resource-manager-template-for-a-windows-virtual-machine-scale-set"></a>使用 Windows 虛擬機器擴展集的 Resource Manager 範本將客體作業系統計量傳送至 Azure 監視器計量存放區

Azure 監視器 [Windows Azure 斬斷擴充功能](azure-diagnostics.md) (WAD) 可讓您從當作虛擬機器、雲端服務或 Service Fabric 叢集一部分執行的客體作業系統 (客體 OS) 收集計量與記錄。  擴充功能可以將遙測資料傳送到先前連結的文章中所列的許多不同位置。  

此文章說明將 Windows 虛擬機器擴展集的客體作業系統效能計量傳送至 Azure 監視器資料存放區的程序。 從 WAD 1.11 版開始，您可以直接將計量寫入到已收集標準平台計量的 Azure 監視器計量存放區。 將它們儲存在此位置可讓您存取與平台計量可用之動作相同的動作。  動作包括近乎即時的警示、圖表、路由、從 REST API 存取以及更多功能。  在過去，WAD 擴充功能會寫入到 Azure 儲存體，而不是 Azure 監視器資料存放區。  

如果您剛開始使用 Resource Manager 範本，請了解[範本部署](../azure-resource-manager/resource-group-overview.md)與其結構和語法。  

## <a name="pre-requisites"></a>先決條件

- 您必須先向 [Microsoft.Insights](https://docs.microsoft.com/powershell/azure/overview?view=azurermps-6.8.1) 註冊您的訂用帳戶 

- 您需要安裝 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview?view=azurermps-6.8.1)，或可以使用 [Azure CloudShell](https://docs.microsoft.com/azure/cloud-shell/overview.md) 


## <a name="set-up-azure-monitor-as-a-data-sink"></a>設定 Azure 監視器作為資料接收器 
Azure 診斷擴充功能會使用稱為「資料接收器」的功能，將計量與記錄路由傳送至不同的位置。  下列步驟示範如何使用新的「Azure 監視器 」資料接收器，使用 Resource Manager 範本與 PowerShell 來部署 VM。 

## <a name="author-resource-manager-template"></a>製作 Resource Manager 範本 
針對此範例，您可以使用公開提供的範例範本。 起始範本位於 https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-windows-autoscale  

- **Azuredeploy.json** 是預先設定的 Resource Manager 範本，可用於部署虛擬機器擴展集

- **Azuredeploy.parameters.json** 是參數檔案，儲存您要為 VM 設定哪些使用者名稱與密碼等的資訊。 在部署期間，Resource Manager 範本會使用此檔案中設定的參數。 

下載這兩個檔案並儲存在本機。 

###  <a name="modify-azuredeployparametersjson"></a>修改 azuredeploy.parameters.json
開啟 *azuredeploy.parameters.json* 檔案 

- 提供您要部署的 **vmSKU** (建議使用 Standard_D2_v3) 
- 指定您要用於虛擬機器擴展集的 **windowsOSVersion** (建議使用 2016-Datacenter) 
- 要使用 **vmssName** 屬性部署的虛擬機器擴展集資源名稱。 例如，*VMSS-WAD-TEST*。    
- 使用 **instanceCount** 屬性，指定您要在虛擬機器擴展集上執行的 VM 數目
- 輸入虛擬機器擴展集的 **adminUsername** 與 **adminPassword** 值。 這些參數可用來從遠端存取擴展集中的 VM。 這些參數可用來從遠端存取 VM。 請勿使用此範本中的資訊以避免 VM 被駭。 Bot 會掃描網際網路，取得公用 Github 存放庫中的使用者名稱與密碼。 很可能會利用這些預設值來測試 VM。 


###  <a name="modify-azuredeployjson"></a>修改 azuredeploy.json
開啟 *azuredeploy.json* 檔案 

新增變數來儲存 Resource Manager 範本中的儲存體帳戶資訊。 在安裝診斷擴充功能期間，您仍必須提供儲存體帳戶。 除了傳送至 Azure 監視器計量存放區之外，還會將診斷設定檔中指定的任何記錄和/或效能計數器寫入指定的儲存體帳戶。 

```json
"variables": { 
//add this line       
"storageAccountName": "[concat('storage', uniqueString(resourceGroup().id))]", 
 ```
 
在 resources 區段中找到虛擬機器擴展集定義，然後將 "identity" 區段新增至設定。 這可確保 Azure 對其指派系統身分識別。 這個步驟可確保擴展集中的 VM 可對 Azure 監視器發出與本身相關的客體計量。  

```json
  { 
      "type": "Microsoft.Compute/virtualMachineScaleSets", 
      "name": "[variables('namingInfix')]", 
      "location": "[resourceGroup().location]", 
      "apiVersion": "2017-03-30", 
      //add these lines below
      "identity": { 
           "type": "systemAssigned" 
       }, 
       //end of lines to add
 ```

在虛擬機器擴展集資源中，找到 **virtualMachineProfile** 區段。 新增名為 **extensionsProfile** 的新設定檔來管理擴充功能。  


在 **extensionProfile** 中，將新擴充功能新增至範本，如 **VMSS-WAD-extension 區段**所示。  此節是受控服務識別 (MSI) 擴充功能，可確保 Azure 監視器會接受所發出的計量。 **name** 欄位可以包含任何名稱。 

MSI 擴充功能中的下列程式碼也可以將診斷擴充功能與設定作為擴充功能資源，新增至虛擬機器擴展集資源。 請視需要新增/移除效能計數器。 

```json
          "extensionProfile": { 
            "extensions": [ 
            // BEGINNING of added code  
            // Managed service identity   
                { 
                 "name": "VMSS-WAD-extension", 
                 "properties": { 
                       "publisher": "Microsoft.ManagedIdentity", 
                       "type": "ManagedIdentityExtensionForWindows", 
                       "typeHandlerVersion": "1.0", 
                       "autoUpgradeMinorVersion": true, 
                       "settings": { 
                             "port": 50342 
                           }, 
                       "protectedSettings": {} 
                     } 
                                
            }, 
            // add diagnostic extension. (Remove this comment after pasting.)
            { 
              "name": "[concat('VMDiagnosticsVmExt','_vmNodeType0Name')]", 
              "properties": { 
                   "type": "IaaSDiagnostics", 
                   "autoUpgradeMinorVersion": true, 
                   "protectedSettings": { 
                        "storageAccountName": "[variables('storageAccountName')]", 
                        "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')),'2015-05-01-preview').key1]", 
                        "storageAccountEndPoint": "https://core.windows.net/" 
                   }, 
                   "publisher": "Microsoft.Azure.Diagnostics", 
                   "settings": { 
                        "WadCfg": { 
                              "DiagnosticMonitorConfiguration": { 
                                   "overallQuotaInMB": "50000", 
                                   "PerformanceCounters": { 
                                       "scheduledTransferPeriod": "PT1M", 
                                       "sinks": "AzMonSink", 
                                       "PerformanceCounterConfiguration": [ 
                                          { 
              "counterSpecifier": "\\Memory\\% Committed Bytes In Use", 
                                              "sampleRate": "PT15S" 
           }, 
           { 
              "counterSpecifier": "\\Memory\\Available Bytes", 
              "sampleRate": "PT15S" 
           }, 
           { 
              "counterSpecifier": "\\Memory\\Committed Bytes", 
              "sampleRate": "PT15S" 
           } 
                                       ] 
                                 }, 
                                 "EtwProviders": { 
                                       "EtwEventSourceProviderConfiguration": [ 
                                           { 
                                              "provider": "Microsoft-ServiceFabric-Actors", 
                                              "scheduledTransferKeywordFilter": "1", 
                                              "scheduledTransferPeriod": "PT5M", 
                                              "DefaultEvents": { 
                                              "eventDestination": "ServiceFabricReliableActorEventTable" 
                                           } 
                                           }, 
                                           { 
                                              "provider": "Microsoft-ServiceFabric-Services", 
                                              "scheduledTransferPeriod": "PT5M", 
                                              "DefaultEvents": { 
                                                   "eventDestination": "ServiceFabricReliableServiceEventTable" 
                                              } 
                                           } 
                                     ], 
                                     "EtwManifestProviderConfiguration": [ 
                                           { 
                                              "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8", 
                                              "scheduledTransferLogLevelFilter": "Information", 
                                              "scheduledTransferKeywordFilter": "4611686018427387904", 
                                              "scheduledTransferPeriod": "PT5M", 
                                              "DefaultEvents": { 
                                                   "eventDestination": "ServiceFabricSystemEventTable" 
                                              } 
                                          } 
                                     ] 
                               } 
                               }, 
                               "SinksConfig": { 
                                     "Sink": [ 
                                          { 
                                              "name": "AzMonSink", 
                                              "AzureMonitor": {} 
                                          } 
                                      ] 
                               } 
                         }, 
                         "StorageAccount": "[variables('storageAccountName')]" 
                         }, 
                        "typeHandlerVersion": "1.11" 
                  } 
           } 
            ] 
          }
          }
      }
    },
    //end of added code plus a few brackets. Be sure that the number and type of brackets match properly when done. 
    {
      "type": "Microsoft.Insights/autoscaleSettings",
...
```


為儲存體帳戶新增 dependsOn，確保它會以正確的順序建立。 
```json
"dependsOn": [ 
"[concat('Microsoft.Network/loadBalancers/', variables('loadBalancerName'))]", 
"[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]" 
//add this line below
"[concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]" 
 ```

如果尚未在範本中建立儲存體帳戶，請予以建立。  
```json
"resources": [
  // add this code    
  {
     "type": "Microsoft.Storage/storageAccounts",
     "name": "[variables('storageAccountName')]",
     "apiVersion": "2015-05-01-preview",
     "location": "[resourceGroup().location]",
     "properties": {
       "accountType": "Standard_LRS"
      }
  },
  // end added code
  { 
    "type": "Microsoft.Network/virtualNetworks",
    "name": "[variables('virtualNetworkName')]",
```

儲存並關閉這兩個檔案。 

## <a name="deploy-the-resource-manager-template"></a>部署 Resource Manager 範本 

> [!NOTE]
> 您必須執行 Azure 診斷擴充功能 1.5 版或更新版本，並將 Resource Manager 範本中的 "autoUpgradeMinorVersion": 屬性集合設定為 *true*。  接著，Azure 會在啟動 VM 時載入適當的擴充功能。 如果您的範本中沒有這些設定，請進行變更，並重新部署該範本。 


我們會利用 Azure PowerShell 來部署 Resource Manager 範本。  

1. 啟動 PowerShell 
1. 使用 `Login-AzureRmAccount` 登入 Azure
1. 使用 `Get-AzureRmSubscription` 取得訂用帳戶清單
1. 設定您會建立/更新當中虛擬機器的訂用帳戶 

   ```PowerShell
   Select-AzureRmSubscription -SubscriptionName "<Name of the subscription>" 
   ```
1. 為要部署的 VM 建立新的資源群組，請執行下列命令 

   ```PowerShell
    New-AzureRmResourceGroup -Name "VMSSWADtestGrp" -Location "<Azure Region>" 
   ```

   > [!NOTE]  
   > 請記得，使用為自訂計量啟用的 Azure 區域。 請參閱 
 
1. 執行下列命令以部署 VM  
   > [!NOTE] 
   > 如果您想要更新現有擴展集，只要將 *-Mode Incremental* 新增至下列命令的結尾。 
 
   ```PowerShell
   New-AzureRmResourceGroupDeployment -Name "VMSSWADTest" -ResourceGroupName "VMSSWADtestGrp" -TemplateFile "<File path of your azuredeploy.JSON file>" -TemplateParameterFile "<File path of your azuredeploy.parameters.JSON file>"  
   ```

1. 部署成功後，您應該能在 Azure 入口網站中找到該虛擬機器擴展集，而且應正在向 Azure 監視器發出計量。 

   > [!NOTE] 
   > 您可能會遇到和所選 vmSkuSize 相關的錯誤。 如果發生這種情況，請回到 azuredeploy.json 檔案，然後更新 vmSkuSize 參數的預設值。 在此案例中，建議您嘗試 "Standard_DS1_v2"。 


## <a name="chart-your-metrics"></a>繪製計量圖表 

1. 登入 Azure 入口網站 

1. 在左側功能表中，按一下 [監視] 

1. 在 [監視] 頁面上，按一下 [計量]。 

   ![[計量] 頁面](./media/metrics-store-custom-rest-api/metrics.png) 

1. 將彙總期間變更為 [過去 30 分鐘]。  

1. 在 [資源] 下拉式清單中，選取您剛才建立的虛擬機器擴展集。  

1. 在 [命名空間] 下拉式清單中，選取 **azure.vm.windows.guest** 

1. 在 [計量] 下拉式清單中，選取 **記憶體\%認可的位元組 (使用中)**。  

接著，您也可以選擇這個計量上的維度，為擴展集中特定的 VM 繪製此計量的圖表，或對擴展集中的每個 VM 都繪製。 



## <a name="next-steps"></a>後續步驟
- 深入了解[自訂計量](metrics-custom-overview.md)。

