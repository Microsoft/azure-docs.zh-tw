---
title: Azure Resource Manager 範本輸出 | Microsoft Docs
description: 描述如何使用宣告式 JSON 語法來定義 Azure Resource Manager 範本的輸出。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/14/2017
ms.author: tomfitz
ms.openlocfilehash: e3c5a581b02f1dd7b7415ebd93de0e425ac2f8ae
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34358360"
---
# <a name="outputs-section-in-azure-resource-manager-templates"></a>Azure Resource Manager 範本中的 Outputs 區段
在輸出區段中，您可以指定從部署傳回的值。 例如，您可以傳回 URI 以存取所部署的資源。

## <a name="define-and-use-output-values"></a>定義和使用輸出值

下列範例示範如何傳回公用 IP 位址的資源識別碼：

```json
"outputs": {
  "resourceID": {
    "type": "string",
    "value": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIPAddresses_name'))]"
  }
}
```

部署之後，您就可以使用指令碼來擷取值。 對於 PowerShell，請使用：

```powershell
(Get-AzureRmResourceGroupDeployment -ResourceGroupName <resource-group-name> -Name <deployment-name>).Outputs.resourceID.value
```

對於 Azure CLI，請使用：

```azurecli-interactive
az group deployment show -g <resource-group-name> -n <deployment-name> --query properties.outputs.resourceID.value
```

您也可以使用 [reference](resource-group-template-functions-resource.md#reference) 函式，從連結的範本中擷取輸出值。 若要從連結的範本取得輸出值，請使用如下語法擷取屬性值：`"[reference('<name-of-deployment>').outputs.<property-name>.value]"`。

例如，您可以藉由從連結的範本中擷取值，在負載平衡器上設定 IP 位址。

```json
"publicIPAddress": {
    "id": "[reference('linkedTemplate').outputs.resourceID.value]"
}
```

您無法在[巢狀範本](resource-group-linked-templates.md#link-or-nest-a-template)的 outputs 區段中使用 `reference` 函式。 若要傳回巢狀範本中已部署資源的值，請將巢狀範本轉換成連結的範本。

## <a name="available-properties"></a>可用屬性

下列範例顯示輸出定義的結構：

```json
"outputs": {
    "<outputName>" : {
        "type" : "<type-of-output-value>",
        "value": "<output-value-expression>"
    }
}
```

| 元素名稱 | 必要 | 說明 |
|:--- |:--- |:--- |
| outputName |yes |輸出值的名稱。 必須是有效的 JavaScript 識別碼。 |
| type |yes |輸出值的類型。 輸出值支援與範本輸入參數相同的類型。 |
| value |yes |評估並傳回做為輸出值的範本語言運算式。 |

## <a name="recommendations"></a>建議

如果您使用範本來建立公用 IP 位址，請包含 outputs 區段，以傳回 IP 位址和完整網域名稱 (FQDN) 的詳細資料。 您可以使用輸出值，輕鬆在部署後擷取公用 IP 位址和 FQDN 的相關詳細資料。

```json
"outputs": {
    "fqdn": {
        "value": "[reference(parameters('publicIPAddresses_name')).dnsSettings.fqdn]",
        "type": "string"
    },
    "ipaddress": {
        "value": "[reference(parameters('publicIPAddresses_name')).ipAddress]",
        "type": "string"
    }
}
```

## <a name="example-templates"></a>範本的範例


|範本  |說明  |
|---------|---------|
|[複製變數](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/copyvariables.json) | 建立複雜的變數，並輸出那些值。 不會部署任何資源。 |
|[公用 IP 位址](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip.json) | 建立公用 IP 位址，並輸出資源識別碼。 |
|[負載平衡器](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip-parentloadbalancer.json) | 上述範本的連結。 建立負載平衡器時，在輸出中使用資源識別碼。 |


## <a name="next-steps"></a>後續步驟
* 若要檢視許多不同類型解決方案的完整範本，請參閱 [Azure 快速入門範本](https://azure.microsoft.com/documentation/templates/)。
* 如需您可以在範本內使用哪些函式的詳細資料，請參閱 [Azure Resource Manager 範本函式](resource-group-template-functions.md)。
* 若要在部署期間合併多個範本，請參閱 [透過 Azure Resource Manager 使用連結的範本](resource-group-linked-templates.md)。
* 您可能需要使用不同資源群組內的資源。 這案例常見於使用多個資源群組之間所共用的儲存體帳戶或虛擬網路時。 如需詳細資訊，請參閱 [resourceId 函式](resource-group-template-functions-resource.md#resourceid)。