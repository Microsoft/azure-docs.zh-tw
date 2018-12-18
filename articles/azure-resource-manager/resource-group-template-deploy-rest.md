---
title: 使用 REST API 和範本部署資源 | Microsoft Docs
description: 使用 Azure Resource Manager 和 Resource Manager REST API 將資源部署到 Azure。 資源會定義在 Resource Manager 範本中。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: 1d8fbd4c-78b0-425b-ba76-f2b7fd260b45
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/01/2018
ms.author: tomfitz
ms.openlocfilehash: ae2393d16d2c9c1000b00f5514e63c988303a83c
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39628506"
---
# <a name="deploy-resources-with-resource-manager-templates-and-resource-manager-rest-api"></a>使用 Resource Manager 範本和 Resource Manager REST API 部署資源

這篇文章說明如何使用 Resource Manager REST API 與 Resource Manager 範本來將您的資源部署至 Azure。  

> [!TIP]
> 如需部署期間偵錯錯誤的說明，請參閱︰
> 
> * [檢視部署作業](resource-manager-deployment-operations.md)，以了解有關取得可協助您針對錯誤進行疑難排解的資訊
> * [針對使用 Azure Resource Manager 將資源部署至 Azure 時常見的錯誤進行疑難排解](resource-manager-common-deployment-errors.md) ，以了解如何解決常見的部署錯誤
> 
> 

您的範本可以是本機檔案，或者是透過 URI 提供使用的外部檔案。 當您的範本位於儲存體帳戶中時，您可以限制範本的存取權，並在部署期間提供共用存取簽章 (SAS) Token

## <a name="deploy-with-the-rest-api"></a>使用 REST API 部署
1. 設定[一般參數和標頭](/rest/api/azure/) (包括驗證權杖)。

2. 如果您沒有現有資源群組，請建立新的資源群組。 提供您的訂用帳戶識別碼、新資源群組的名稱，以及需要解決方案的位置。 如需詳細資訊，請參閱[建立資源群組](/rest/api/resources/resourcegroups/createorupdate)。

  ```HTTP
  PUT https://management.azure.com/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>?api-version=2015-01-01
  {
    "location": "West US",
    "tags": {
      "tagname1": "tagvalue1"
    }
  }
  ```

3. 請先執行[驗證範本部署作業](/rest/api/resources/deployments/validate)來驗證部署，然後再執行部署。 測試部署時，請提供與執行部署時完全一致的參數 (如下個步驟所示)。

4. 建立部署。 提供您的訂用帳戶識別碼、資源群組的名稱、部署的名稱，以及範本的連結。 如需範本檔案的相關資訊，請參閱[參數檔案](#parameter-file)。 如需以 REST API 建立資源群組的相關詳細資訊，請參閱[建立範本部署](https://docs.microsoft.com/rest/api/resources/deployments#Deployments_CreateOrUpdate)。 請注意，**mode** 是設為 **Incremental**。 若要執行完整部署，請將 **mode** 設為 **Complete**。 使用完整模式時請務必謹慎，因為您可能會不小心刪除不在範本中的資源。

  ```HTTP
  PUT https://management.azure.com/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>/providers/Microsoft.Resources/deployments/<YourDeploymentName>?api-version=2015-01-01
  {
    "properties": {
      "templateLink": {
        "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json",
        "contentVersion": "1.0.0.0"
      },
      "mode": "Incremental",
      "parametersLink": {
        "uri": "http://mystorageaccount.blob.core.windows.net/templates/parameters.json",
        "contentVersion": "1.0.0.0"
      }
    }
  }
  ```

    如果您想要記錄回應內容及/或要求內容，可在要求中包括 **debugSetting** 。

  ```HTTP
  "debugSetting": {
    "detailLevel": "requestContent, responseContent"
  }
  ```

    您可以設定儲存體帳戶以使用共用存取簽章 (SAS) Token。 如需詳細資訊，請參閱[使用共用存取簽章委派存取](https://docs.microsoft.com/rest/api/storageservices/delegating-access-with-a-shared-access-signature)。

5. 取得範本部署的狀態。 如需詳細資訊，請參閱[取得範本部署的相關資訊](/rest/api/resources/deployments/get)。

  ```HTTP
  GET https://management.azure.com/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>/providers/Microsoft.Resources/deployments/<YourDeploymentName>?api-version=2015-01-01
  ```

## <a name="redeploy-when-deployment-fails"></a>部署失敗時重新部署

針對失敗的部署，您可以將部署歷程記錄中先前的部署指定為自動進行重新部署。 若要使用這個選項，您的部署必須有唯一的名稱，以便在歷程記錄中進行識別。 如果您沒有唯一的名稱，則目前失敗的部署可能會覆寫歷程記錄中先前成功的部署。 您只可以使用此選項搭配根層級部署。 從巢狀範本部署不適用於重新部署。

如果目前的部署失敗，若要重新部署最後一個成功的部署，請使用：

```HTTP
"onErrorDeployment": {
  "type": "LastSuccessful",
},
```

如果目前的部署失敗，若要重新部署特定部署，請使用：

```HTTP
"onErrorDeployment": {
  "type": "SpecificDeployment",
  "deploymentName": "<deploymentname>"
}
```

指定的部署必須成功。

## <a name="parameter-file"></a>參數檔案

如果您在部署期間使用參數檔案傳遞參數值，您必須使用類似於下列範例的格式建立 JSON 檔案：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "webSiteName": {
            "value": "ExampleSite"
        },
        "webSiteHostingPlanName": {
            "value": "DefaultPlan"
        },
        "webSiteLocation": {
            "value": "West US"
        },
        "adminPassword": {
            "reference": {
               "keyVault": {
                  "id": "/subscriptions/{guid}/resourceGroups/{group-name}/providers/Microsoft.KeyVault/vaults/{vault-name}"
               }, 
               "secretName": "sqlAdminPassword" 
            }   
        }
   }
}
```

參數檔的大小不得超過 64 KB。

如果您需要提供參數機密的值 (例如密碼)，請將該值加入金鑰保存庫。 在部署期間擷取金鑰保存庫，如先前範例所示。 如需詳細資訊，請參閱 [在部署期間傳遞安全值](resource-manager-keyvault-parameter.md)。 

## <a name="next-steps"></a>後續步驟
* 若要指定如何處理存在於資源群組中但尚未定義於範本中的資源，請參閱 [Azure Resource Manager 部署模式](deployment-modes.md)。
* 若要了解如何處理非同步 REST 作業，請參閱[追蹤非同步 Azure 作業](resource-manager-async-operations.md) \(英文\)。
* 如需透過 .NET 用戶端程式庫部署資源的範例，請參閱 [使用 .NET 程式庫與範本部署資源](../virtual-machines/windows/csharp-template.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
* 若要在範本中定義參數，請參閱 [編寫範本](resource-group-authoring-templates.md#parameters)。
* 如需關於企業如何使用 Resource Manager 有效地管理訂用帳戶的指引，請參閱 [Azure 企業 Scaffold - 規定的訂用帳戶治理](/azure/architecture/cloud-adoption-guide/subscription-governance)。

