---
title: Azure 父代資源錯誤 | Microsoft Docs
description: 描述當使用父代資源時如何解決錯誤。
services: azure-resource-manager
documentationcenter: ''
author: tfitzmac
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 08/01/2018
ms.author: tomfitz
ms.openlocfilehash: 3042ea1a523f12ae0311545a1b9bc67306f266dd
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39447297"
---
# <a name="resolve-errors-for-parent-resources"></a>解決父代資源的錯誤

本文說明當部署相依於父代資源的資源時，可能會發生錯誤。

## <a name="symptom"></a>徵狀

部署是另一個資源子系的資源時，您可能會遇到下列錯誤：

```
Code=ParentResourceNotFound;
Message=Can not perform requested operation on nested resource. Parent resource 'exampleserver' not found."
```

## <a name="cause"></a>原因

當某資源為另一個資源的子系資源時，父代資源必須在建立子系資源之前就存在。 子資源的名稱會定義與父代名稱的連線。 子資源名稱的格式如下︰`<parent-resource-name>/<child-resource-name>`。 例如，SQL Database 可能定義如下︰

```json
{
  "type": "Microsoft.Sql/servers/databases",
  "name": "[concat(variables('databaseServerName'), '/', parameters('databaseName'))]",
  ...
```

如果您在相同的範本中同時部署伺服器和資料庫，但未在伺服器上指定相依性，則可在部署伺服器之前啟動資料庫部署。 

如果父代資源已經存在，但未部署在相同的範本中，當 Resource Manager 無法讓子資源與父代產生關聯時，您會發生此錯誤。 子資源不是正確的格式，或子資源部署與父代資源資源群組不同的資源群組時，可能會發生此錯誤。

## <a name="solution"></a>解決方法

當父代和子資源部署在相同的範本時，若要解決這個錯誤，請包含相依性。

```json
"dependsOn": [
    "[variables('databaseServerName')]"
]
```

當父代資源先前部署在不同的範本時，若要解決這個錯誤，請勿設定相依性。 反而，將子系部署至相同的資源群組的，並提供父代資源的名稱。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "sqlServerName": {
            "type": "string"
        },
        "databaseName": {
            "type": "string"
        }
    },
    "resources": [
        {
            "apiVersion": "2014-04-01",
            "type": "Microsoft.Sql/servers/databases",
            "location": "[resourceGroup().location]",
            "name": "[concat(parameters('sqlServerName'), '/', parameters('databaseName'))]",
            "properties": {
                "collation": "SQL_Latin1_General_CP1_CI_AS",
                "edition": "Basic"
            }
        }
    ],
    "outputs": {}
}
```

如需詳細資訊，請參閱[定義 Azure Resource Manager 範本中部署資源的順序](resource-group-define-dependencies.md)。