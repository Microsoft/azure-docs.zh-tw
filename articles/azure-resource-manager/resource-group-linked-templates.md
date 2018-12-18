---
title: 連結 Azure 部署的範本 | Microsoft Docs
description: 描述如何在「Azure 資源管理員」範本中使用連結的範本，以建立模組化範本方案。 示範如何傳遞參數值、指定參數檔案，以及動態建立 URL。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: 27d8c4b2-1e24-45fe-88fd-8cf98a6bb2d2
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/10/2018
ms.author: tomfitz
ms.openlocfilehash: 8cac3c8d3a1877ad7c93efc0954c2f07ecaa0a29
ms.sourcegitcommit: a2ae233e20e670e2f9e6b75e83253bd301f5067c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/13/2018
ms.locfileid: "42141270"
---
# <a name="using-linked-and-nested-templates-when-deploying-azure-resources"></a>部署 Azure 資源時使用連結和巢狀的範本

若要部署解決方案，您可以使用單一範本或有許多相關範本的主要範本。 相關範本可以是連結到主要範本的個別檔案，或是主要範本內巢狀的範本。

若為小型至中型的解決方案，單一範本會比較容易了解和維護。 您可以在單一檔案中看到所有的資源和值。 若為進階案例，連結的範本可讓您將解決方案細分成目標元件，並重複使用範本。

在使用連結的範本時，您會建立主要範本以在部署期間接收參數值。 主要範本包含所有連結的範本，並且會視需要將值傳遞至這些範本。

## <a name="link-or-nest-a-template"></a>連結或巢狀範本

若要連結到其他範本，請在主要範本中新增**部署**資源。

```json
"resources": [
  {
      "apiVersion": "2017-05-10",
      "name": "linkedTemplate",
      "type": "Microsoft.Resources/deployments",
      "properties": {
          "mode": "Incremental",
          <nested-template-or-external-template>
      }
  }
]
```

您針對部署資源所提供的屬性，會根據您是連結到外部範本還是在主要範本中巢狀內嵌範本而有所不同。

針對連結和巢狀的範本，您只可以使用[累加](deployment-modes.md)部署模式。

### <a name="nested-template"></a>巢狀範本

若要在主要範本巢狀範本，使用 **template** 屬性，並指定範本語法。

```json
"resources": [
  {
    "apiVersion": "2017-05-10",
    "name": "nestedTemplate",
    "type": "Microsoft.Resources/deployments",
    "properties": {
      "mode": "Incremental",
      "template": {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
          {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[variables('storageName')]",
            "apiVersion": "2015-06-15",
            "location": "West US",
            "properties": {
              "accountType": "Standard_LRS"
            }
          }
        ]
      }
    }
  }
]
```

> [!NOTE]
> 對於巢狀範本，您無法使用巢狀範本中定義的參數或變數。 您可以使用來自主要範本的參數和變數。 在上述範例中，`[variables('storageName')]` 會從主要範本擷取值，而不是巢狀範本。 這項限制不適用於外部範本。
>
> 您無法在巢狀範本的輸出區段中使用 `reference` 函式。 若要傳回巢狀範本中已部署資源的值，請將巢狀範本轉換成連結範本。

巢狀範本需要與標準範本[相同的屬性](resource-group-authoring-templates.md)。

### <a name="external-template-and-external-parameters"></a>外部範本和外部參數

若要連結到外部的範本和參數檔案，請使用 **templateLink** 和 **parametersLink**。 在連結到範本時，Resource Manager 服務必須能夠存取該範本。 您無法指定本機檔案或只可在本機網路存取的檔案。 您可以只提供 URI 值，其中包含 **http** 或 **https**。 有一個選項是將連結的範本放在儲存體帳戶中，並將 URI 用於該項目。

```json
"resources": [
  {
     "apiVersion": "2017-05-10",
     "name": "linkedTemplate",
     "type": "Microsoft.Resources/deployments",
     "properties": {
       "mode": "incremental",
       "templateLink": {
          "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.json",
          "contentVersion":"1.0.0.0"
       },
       "parametersLink": {
          "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.parameters.json",
          "contentVersion":"1.0.0.0"
       }
     }
  }
]
```

您不需要提供範本的 `contentVersion` 屬性或參數。 如果您未提供內容版本值，則會部署目前的版本範本。 如果您提供內容版本值，它必須符合所連結範本中的版本；否則，部署會因為錯誤而失敗。

### <a name="external-template-and-inline-parameters"></a>外部範本和內嵌參數

或者，您也可以提供內嵌的參數。 若要將值從主要範本傳遞至連結的範本，請使用 **parameters**。

```json
"resources": [
  {
     "apiVersion": "2017-05-10",
     "name": "linkedTemplate",
     "type": "Microsoft.Resources/deployments",
     "properties": {
       "mode": "incremental",
       "templateLink": {
          "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.json",
          "contentVersion":"1.0.0.0"
       },
       "parameters": {
          "StorageAccountName":{"value": "[parameters('StorageAccountName')]"}
        }
     }
  }
]
```

## <a name="using-variables-to-link-templates"></a>使用變數來連結範本

先前的範例示範了範本連結的硬式編碼 URL 值。 這種方法可能適用於簡單的範本，但不適合一組大型的模組化範本。 不過，您可以建立一個儲存主要範本之基底 URL 的靜態變數，然後再從該基底 URL 連結動態建立連結的範本之 URL。 這個方法的好處是您可以輕鬆地移動範本或建立分支範本，因為您只需要變更主要範本中的靜態變數。 主要範本會於整個分解的範本傳遞正確的 URI。

下列範例示範如何使用基底 URL 來為連結的範本 (**sharedTemplateUrl** 和 **vmTemplate**) 建立兩個 URL。

```json
"variables": {
    "templateBaseUrl": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/postgresql-on-ubuntu/",
    "sharedTemplateUrl": "[concat(variables('templateBaseUrl'), 'shared-resources.json')]",
    "vmTemplateUrl": "[concat(variables('templateBaseUrl'), 'database-2disk-resources.json')]"
}
```

您也可以使用 [deployment()](resource-group-template-functions-deployment.md#deployment) 取得目前範本的基底 URL，用來取得相同位置中其他範本的 URL。 如果您的範本位置變更，或您想要避免在範本檔案中使用硬式編碼 URL，這十分實用。 只有在透過 URL 連結至遠端範本時，才會傳回 templateLink 屬性。 如果您使用本機範本，就無法使用該屬性。

```json
"variables": {
    "sharedTemplateUrl": "[uri(deployment().properties.templateLink.uri, 'shared-resources.json')]"
}
```

## <a name="get-values-from-linked-template"></a>從連結的範本取得值

若要從連結的範本取得輸出值，請使用如下語法擷取屬性值：`"[reference('<name-of-deployment>').outputs.<property-name>.value]"`。

下列範例會示範如何參考連結的範本，並擷取輸出值。 連結的範本會傳回簡單的訊息。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [],
    "outputs": {
        "greetingMessage": {
            "value": "Hello World",
            "type" : "string"
        }
    }
}
```

主要範本會部署連結的範本，並取得傳回的值。 請注意，它會依名稱參考部署資源，並使用連結的範本所傳回之屬性的名稱。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [
        {
            "apiVersion": "2017-05-10",
            "name": "linkedTemplate",
            "type": "Microsoft.Resources/deployments",
            "properties": {
                "mode": "incremental",
                "templateLink": {
                    "uri": "[uri(deployment().properties.templateLink.uri, 'helloworld.json')]",
                    "contentVersion": "1.0.0.0"
                }
            }
        }
    ],
    "outputs": {
        "messageFromLinkedTemplate": {
            "type": "string",
            "value": "[reference('linkedTemplate').outputs.greetingMessage.value]"
        }
    }
}
```

如同其他資源類型，您可以設定連結的範本和其他資源之間的相依性。 因此，當其他資源需要來自所連結範本的輸出值時，請確定在資源之前部署所連結的範本。 或者，當連結的範本仰賴其他資源時，請確定在所連結的範本之前部署其他資源。

下列範例所顯示的範本，會部署公用 IP 位址並傳回資源識別碼：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "publicIPAddresses_name": {
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "name": "[parameters('publicIPAddresses_name')]",
            "apiVersion": "2017-06-01",
            "location": "eastus",
            "properties": {
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Dynamic",
                "idleTimeoutInMinutes": 4
            },
            "dependsOn": []
        }
    ],
    "outputs": {
        "resourceID": {
            "type": "string",
            "value": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIPAddresses_name'))]"
        }
    }
}
```

若要在部署負載平衡器時，使用上述範本中的公用 IP 位址，請連結到該範本並對部署資源新增相依性。 負載平衡器上的公用 IP 位址會設定為從連結的範本傳回的輸出值。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "loadBalancers_name": {
            "defaultValue": "mylb",
            "type": "string"
        },
        "publicIPAddresses_name": {
            "defaultValue": "myip",
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/loadBalancers",
            "name": "[parameters('loadBalancers_name')]",
            "apiVersion": "2017-06-01",
            "location": "eastus",
            "properties": {
                "frontendIPConfigurations": [
                    {
                        "name": "LoadBalancerFrontEnd",
                        "properties": {
                            "privateIPAllocationMethod": "Dynamic",
                            "publicIPAddress": {
                                "id": "[reference('linkedTemplate').outputs.resourceID.value]"
                            }
                        }
                    }
                ],
                "backendAddressPools": [],
                "loadBalancingRules": [],
                "probes": [],
                "inboundNatRules": [],
                "outboundNatRules": [],
                "inboundNatPools": []
            },
            "dependsOn": [
                "linkedTemplate"
            ]
        },
        {
            "apiVersion": "2017-05-10",
            "name": "linkedTemplate",
            "type": "Microsoft.Resources/deployments",
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                    "uri": "[uri(deployment().properties.templateLink.uri, 'publicip.json')]",
                    "contentVersion": "1.0.0.0"
                },
                "parameters":{
                    "publicIPAddresses_name":{"value": "[parameters('publicIPAddresses_name')]"}
                }
            }
        }
    ]
}
```

## <a name="linked-and-nested-templates-in-deployment-history"></a>部署歷程記錄中連結和巢狀的範本

在部署歷程記錄中，Resource Manager 會以個別部署的方式處理每一個範本。 因此，具有三個連結或巢狀範本的主要範本在部署歷程記錄中會顯示為：

![部署歷程記錄](./media/resource-group-linked-templates/deployment-history.png)

您可以使用歷程記錄中的這些個別項目，以在部署後擷取輸出值。 下列範本會建立公用 IP 位址，並輸出 IP 位址：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "publicIPAddresses_name": {
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "name": "[parameters('publicIPAddresses_name')]",
            "apiVersion": "2017-06-01",
            "location": "southcentralus",
            "properties": {
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Static",
                "idleTimeoutInMinutes": 4,
                "dnsSettings": {
                    "domainNameLabel": "[concat(parameters('publicIPAddresses_name'), uniqueString(resourceGroup().id))]"
                }
            },
            "dependsOn": []
        }
    ],
    "outputs": {
        "returnedIPAddress": {
            "type": "string",
            "value": "[reference(parameters('publicIPAddresses_name')).ipAddress]"
        }
    }
}
```

下列範本會連結至先前的範本。 它會建立三個公用 IP 位址。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
    },
    "variables": {},
    "resources": [
        {
            "apiVersion": "2017-05-10",
            "name": "[concat('linkedTemplate', copyIndex())]",
            "type": "Microsoft.Resources/deployments",
            "properties": {
              "mode": "Incremental",
              "templateLink": {
                "uri": "[uri(deployment().properties.templateLink.uri, 'static-public-ip.json')]",
                "contentVersion": "1.0.0.0"
              },
              "parameters":{
                  "publicIPAddresses_name":{"value": "[concat('myip-', copyIndex())]"}
              }
            },
            "copy": {
                "count": 3,
                "name": "ip-loop"
            }
        }
    ]
}
```

在部署後，您可以使用下列 PowerShell 指令碼擷取輸出值：

```powershell
$loopCount = 3
for ($i = 0; $i -lt $loopCount; $i++)
{
    $name = 'linkedTemplate' + $i;
    $deployment = Get-AzureRmResourceGroupDeployment -ResourceGroupName examplegroup -Name $name
    Write-Output "deployment $($deployment.DeploymentName) returned $($deployment.Outputs.returnedIPAddress.value)"
}
```

或者，Azure CLI 指令碼：

```azurecli
for i in 0 1 2;
do
    name="linkedTemplate$i";
    deployment=$(az group deployment show -g examplegroup -n $name);
    ip=$(echo $deployment | jq .properties.outputs.returnedIPAddress.value);
    echo "deployment $name returned $ip";
done
```

## <a name="securing-an-external-template"></a>保護外部範本

雖然連結的範本必須可從外部存取，但它不必供大眾存取。 您可以將您的範本新增至只有儲存體帳戶擁有者可以存取的私人儲存體帳戶。 接著，在部署期間建立共用存取簽章 (SAS) Token 來啟用存取權。 您會將該 SAS Token 加入連結範本的 URI。 即使該 Token 是以安全字串來傳遞，連結範本的 URI (包括 SAS Token) 還是會記錄在部署作業中。 為了限制公開的程度，請為該 Token 設定到期日。

當然，也可以將參數檔案限制為透過 SAS Token 存取。

下列範例顯示如何在連結到範本時傳遞 SAS 權杖：

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "containerSasToken": { "type": "string" }
  },
  "resources": [
    {
      "apiVersion": "2017-05-10",
      "name": "linkedTemplate",
      "type": "Microsoft.Resources/deployments",
      "properties": {
        "mode": "incremental",
        "templateLink": {
          "uri": "[concat(uri(deployment().properties.templateLink.uri, 'helloworld.json'), parameters('containerSasToken'))]",
          "contentVersion": "1.0.0.0"
        }
      }
    }
  ],
  "outputs": {
  }
}
```

在 PowerShell 中，您會取得容器的 Token 並使用下列命令部署範本。 請注意，**containerSasToken** 參數會定義於範本中。 它不是 **New-AzureRmResourceGroupDeployment** 命令中的參數。

```powershell
Set-AzureRmCurrentStorageAccount -ResourceGroupName ManageGroup -Name storagecontosotemplates
$token = New-AzureStorageContainerSASToken -Name templates -Permission r -ExpiryTime (Get-Date).AddMinutes(30.0)
$url = (Get-AzureStorageBlob -Container templates -Blob parent.json).ICloudBlob.uri.AbsoluteUri
New-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateUri ($url + $token) -containerSasToken $token
```

在 Azure CLI 中，您會取得容器的 Token 並使用下列指令碼部署範本：

```azurecli
expiretime=$(date -u -d '30 minutes' +%Y-%m-%dT%H:%MZ)
connection=$(az storage account show-connection-string \
    --resource-group ManageGroup \
    --name storagecontosotemplates \
    --query connectionString)
token=$(az storage container generate-sas \
    --name templates \
    --expiry $expiretime \
    --permissions r \
    --output tsv \
    --connection-string $connection)
url=$(az storage blob url \
    --container-name templates \
    --name parent.json \
    --output tsv \
    --connection-string $connection)
parameter='{"containerSasToken":{"value":"?'$token'"}}'
az group deployment create --resource-group ExampleGroup --template-uri $url?$token --parameters $parameter
```

## <a name="example-templates"></a>範本的範例

下列範例顯示連結範本的一般用途。

|主要的範本  |連結的範本 |說明  |
|---------|---------| ---------|
|[Hello World](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/helloworldparent.json) |[連結的範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/helloworld.json) | 從連結的範本傳回字串。 |
|[使用公用 IP 位址的負載平衡器](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip-parentloadbalancer.json) |[連結的範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip.json) |從連結的範本傳回公用 IP 位址，並且在負載平衡器中設定該值。 |
|[多個 IP 位址](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/static-public-ip-parent.json) | [連結的範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/static-public-ip.json) |在連結的範本中建立數個公用 IP 位址。  |

## <a name="next-steps"></a>後續步驟

* 若要了解如何定義您資源的部署順序，請參閱 [定義 Azure Resource Manager 範本中的相依性](resource-group-define-dependencies.md)。
* 若要了解如何定義一個資源，但建立它的多個執行個體，請參閱 [在 Azure Resource Manager 中建立資源的多個執行個體](resource-group-create-multiple.md)。
* 如需在儲存體帳戶中設定範本並產生 SAS Token 的步驟，請參閱[使用 Resource Manager 範本與 Azure PowerShell 部署資源](resource-group-template-deploy.md)或 [Deploy resources with Resource Manager templates and Azure CLI (使用 Resource Manager 範本和 Azure CLI 部署資源)](resource-group-template-deploy-cli.md)。