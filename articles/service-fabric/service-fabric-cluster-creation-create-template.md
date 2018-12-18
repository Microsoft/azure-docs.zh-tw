---
title: 建立 Azure Service Fabric 叢集範本 | Microsoft Docs
description: 了解如何為 Service Fabric 叢集建立 Resource Manager 範本。 為用戶端驗證設定安全性、Azure Key Vault 與 Azure Active Directory (Azure AD)。
services: service-fabric
documentationcenter: .net
author: aljo-microsoft
manager: timlt
editor: chackdan
ms.assetid: 15d0ab67-fc66-4108-8038-3584eeebabaa
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/16/2018
ms.author: aljo
ms.openlocfilehash: 9482b1a33caaf73838101431dfc1faac7020ee42
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "40234689"
---
# <a name="create-a-service-fabric-cluster-resource-manager-template"></a>建立 Service Fabric 叢集 Resource Manager 範本

[Azure Service Fabric 叢集](service-fabric-deploy-anywhere.md)是一組透過網路連線的虛擬機器，您可以將微服務部署到其中並進行管理。 在 Azure 中執行的 Service Fabric 叢集是 Azure 資源，而且是使用 Resource Manager 來部署、管理及監視的。  此文章說明如何為在 Azure 中執行的 Service Fabric 叢集建立 Resource Manager 範本。  當範本完成時，您可以[在 Azure 上部署叢集](service-fabric-cluster-creation-via-arm.md)。

叢集安全性是在第一次設定叢集時所設定，而且稍後無法變更。 設定叢集之前，請閱讀 [Service Fabric 叢集安全性案例][service-fabric-cluster-security]。 在 Azure 中，Service Fabric 使用 x509 憑證來保護您的叢集與其端點、驗證用戶端，以及加密資料。 也建議您使用 Azure Active Directory 來保護對管理端點的存取。 必須在建立叢集之前先建立 Azure AD 租用戶與使用者。  如需詳細資訊，請閱讀[設定 Azure AD 以驗證用戶端](service-fabric-cluster-creation-setup-aad.md)。

部署生產叢集以執行生產工作負載之前，請務必先閱讀[生產整備檢查清單](service-fabric-production-readiness-checklist.md)。

## <a name="create-the-resource-manager-template"></a>建立 Resource Manager 範本
您可以在 [GitHub 上的 Azure 範例](https://github.com/Azure-Samples/service-fabric-cluster-templates)中取得 Resource Manager 範本範例。 這些範本可以用作叢集範本的起點。

此文章使用 [5 節點安全叢集][service-fabric-secure-cluster-5-node-1-nodetype]範例範本與範本參數。 將 *azuredeploy.json* 與 *azuredeploy.parameters.json* 下載到您的電腦，並在您慣用的文字編輯器中開啟這兩個檔案。

> [!NOTE]
> 針對國家雲 (Azure Government、Azure 中國、Azure 德國)，您也應將下列 `fabricSettings` 新增至您的範本：`AADLoginEndpoint`、`AADTokenEndpointFormat` 和 `AADCertEndpointFormat`。

## <a name="add-certificates"></a>新增憑證
您可以藉由參考包含憑證金鑰的 Key Vault，將憑證新增到叢集 Resource Manager 範本。 在 Resource Manager 範本參數檔案 (*azuredeploy.parameters.json*) 中新增金鑰保存庫參數和值。

### <a name="add-all-certificates-to-the-virtual-machine-scale-set-osprofile"></a>將所有憑證都新增到虛擬機器擴展集 osProfile
安裝在叢集中的每個憑證都必須在擴展集資源 (Microsoft.Compute/virtualMachineScaleSets) 的 [osProfile] 區段中設定妥當。 此動作會指示資源提供者在 VM 上安裝憑證。 此安裝既包含叢集憑證，也包含任何您打算用於應用程式的應用程式安全性憑證︰

```json
{
  "apiVersion": "[variables('vmssApiVersion')]",
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  ...
  "properties": {
    ...
    "osProfile": {
      ...
      "secrets": [
        {
          "sourceVault": {
            "id": "[parameters('sourceVaultValue')]"
          },
          "vaultCertificates": [
            {
              "certificateStore": "[parameters('clusterCertificateStorevalue')]",
              "certificateUrl": "[parameters('clusterCertificateUrlValue')]"
            },
            {
              "certificateStore": "[parameters('applicationCertificateStorevalue')",
              "certificateUrl": "[parameters('applicationCertificateUrlValue')]"
            },
            ...
          ]
        }
      ]
    }
  }
}
```

### <a name="configure-the-service-fabric-cluster-certificate"></a>設定 Service Fabric 叢集憑證

不論是在 Service Fabric 叢集資源 (Microsoft.ServiceFabric/clusters) 中，還是在虛擬機器擴展集資源中虛擬機器擴展集的 Service Fabric 延伸模組中，都必須設定叢集驗證憑證。 這個安排可讓 Service Fabric 資源提供者設定它，以用於管理端點的叢集驗證和伺服器驗證。

#### <a name="add-the-certificate-information-the-virtual-machine-scale-set-resource"></a>將憑證資訊新增至虛擬機器擴展集資源

```json
{
  "apiVersion": "[variables('vmssApiVersion')]",
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  ...
  "properties": {
    ...
    "virtualMachineProfile": {
      "extensionProfile": {
        "extensions": [
          {
            "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType0Name')]",
            "properties": {
              ...
              "settings": {
                ...
                "certificate": {
                  "commonNames": ["[parameters('certificateCommonName')]"],
                  "x509StoreName": "[parameters('clusterCertificateStoreValue')]"
                },
                ...
              }
            }
          }
        ]
      }
    }
  }
}
```

#### <a name="add-the-certificate-information-to-the-service-fabric-cluster-resource"></a>將憑證資訊新增至 Service Fabric 叢集資源

```json
{
  "apiVersion": "2018-02-01",
  "type": "Microsoft.ServiceFabric/clusters",
  "name": "[parameters('clusterName')]",
  "location": "[parameters('clusterLocation')]",
  "dependsOn": [
    "[concat('Microsoft.Storage/storageAccounts/', variables('supportLogStorageAccountName'))]"
  ],
  "properties": {
    "certificateCommonNames": {
        "commonNames": [
        {
            "certificateCommonName": "[parameters('certificateCommonName')]",
            "certificateIssuerThumbprint": ""
        }
        ],
        "x509StoreName": "[parameters('certificateStoreValue')]"
    },
    ...
  }
}
```

## <a name="add-azure-ad-configuration-to-use-azure-ad-for-client-access"></a>新增 Azure AD 設定以針對用戶端存取使用 Azure AD

您可以藉由參考包含憑證金鑰的金鑰保存庫，將 Azure AD 設定新增到叢集 Resource Manager 範本。 在 Resource Manager 範本參數檔案 (*azuredeploy.parameters.json*) 中新增那些 Azure AD 參數和值。 

> [!NOTE]
> 必須在建立叢集之前先建立 Azure AD 租用戶與使用者。  如需詳細資訊，請閱讀[設定 Azure AD 以驗證用戶端](service-fabric-cluster-creation-setup-aad.md)。

```json
{
  "apiVersion": "2018-02-01",
  "type": "Microsoft.ServiceFabric/clusters",
  "name": "[parameters('clusterName')]",
  ...
  "properties": {
    "certificateCommonNames": {
        "commonNames": [
        {
            "certificateCommonName": "[parameters('certificateCommonName')]",
            "certificateIssuerThumbprint": ""
        }
        ],
        "x509StoreName": "[parameters('certificateStoreValue')]"
    },
    ...
    "azureActiveDirectory": {
      "tenantId": "[parameters('aadTenantId')]",
      "clusterApplication": "[parameters('aadClusterApplicationId')]",
      "clientApplication": "[parameters('aadClientApplicationId')]"
    },
    ...
  }
}
```

## <a name="populate-the-parameter-file-with-the-values"></a>將值填入參數檔案中

最後，請使用金鑰保存庫和 Azure AD PowerShell 命令的輸出值來填入參數檔案。

如果您打算使用 Azure Service Fabric RM PowerShell 模組，則不需要填入叢集憑證資訊。 如果您想讓系統產生自我簽署憑證以確保叢集安全性，只須讓其保持為 Null。 

> [!NOTE]
> 若要 RM 模組拾取及填入這些空白參數值，參數名稱必須符合下列名稱

```json
"clusterCertificateThumbprint": {
    "value": ""
},
"certificateCommonName": {
    "value": ""
},
"clusterCertificateUrlValue": {
    "value": ""
},
"sourceVaultvalue": {
    "value": ""
},
```

如果您使用的是應用程式憑證，或已上傳至金鑰保存庫的現有叢集，您需要取得這項資訊並加以填寫。

RM 模組無法為您產生 Azure AD 組態，因此如果您打算對用戶端存取使用 Azure AD，您必須將其填入。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        ...
        "clusterCertificateStoreValue": {
            "value": "My"
        },
        "clusterCertificateThumbprint": {
            "value": "<thumbprint>"
        },
        "clusterCertificateUrlValue": {
            "value": "https://myvault.vault.azure.net:443/secrets/myclustercert/4d087088df974e869f1c0978cb100e47"
        },
        "applicationCertificateStorevalue": {
            "value": "My"
        },
        "applicationCertificateUrlValue": {
            "value": "https://myvault.vault.azure.net:443/secrets/myapplicationcert/2e035058ae274f869c4d0348ca100f08"
        },
        "sourceVaultvalue": {
            "value": "/subscriptions/<guid>/resourceGroups/mycluster-keyvault/providers/Microsoft.KeyVault/vaults/myvault"
        },
        "aadTenantId": {
            "value": "<guid>"
        },
        "aadClusterApplicationId": {
            "value": "<guid>"
        },
        "aadClientApplicationId": {
            "value": "<guid>"
        },
        ...
    }
}
```

## <a name="test-your-template"></a>測試您的範本
使用下列 PowerShell 命令，以參數檔案來測試 Resource Manager 範本︰

```PowerShell
Test-AzureRmResourceGroupDeployment -ResourceGroupName "myresourcegroup" -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json
```

為避免遇到問題和收到難解的訊息，您可選擇使用 "-Debug"。

```PowerShell
Test-AzureRmResourceGroupDeployment -ResourceGroupName "myresourcegroup" -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json -Debug
```

下圖說明 Key Vault 和 Azure AD 組態在 Resource Manager 範本中發生作用的位置。

![Resource Manager 相依性對應][cluster-security-arm-dependency-map]

## <a name="next-steps"></a>後續步驟
現在您已有叢集範本，請了解如何[將叢集部署到 Azure](service-fabric-cluster-creation-via-arm.md)。  部署生產叢集之前，請先閱讀[生產整備檢查清單](service-fabric-production-readiness-checklist.md)。


<!-- Links -->
[service-fabric-cluster-security]: service-fabric-cluster-security.md
[service-fabric-secure-cluster-5-node-1-nodetype]: https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure
[resource-group-template-deploy]: https://azure.microsoft.com/documentation/articles/resource-group-template-deploy/

<!-- Images -->
[cluster-security-arm-dependency-map]: ./media/service-fabric-cluster-creation-create-template/cluster-security-arm-dependency-map.png
