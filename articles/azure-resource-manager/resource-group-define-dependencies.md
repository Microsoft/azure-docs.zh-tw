---
title: 設定 Azure 資源的部署順序 | Microsoft Docs
description: 說明如何在部署期間，將某個資源設定為相依於另一個資源，確保以正確的順序部署資源。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: ''
ms.assetid: 34ebaf1e-480c-4b4d-9bf6-251bd3f8f2cf
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/05/2018
ms.author: tomfitz
ms.openlocfilehash: 308ab9d35e07c8376fb183c794fcad77a74a1df9
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46295558"
---
# <a name="define-the-order-for-deploying-resources-in-azure-resource-manager-templates"></a>定義 Azure Resource Manager 範本中部署資源的順序
針對指定的資源，可能會有部署資源之前必須存在的其他資源。 例如，SQL Server 必須存在，才能嘗試部署 SQL 資料庫。 您可以將一個資源標示為相依於其他資源，來定義此關聯性。 您可以使用 **dependsOn** 元素或使用 **reference** 函式定義相依性。 

資源管理員會評估資源之間的相依性，並依其相依順序進行部署。 如果資源並未彼此相依，Resource Manager 就會平行部署資源。 您只需要針對部署在相同範本中的資源定義相依性。 

如需教學課程，請參閱[教學課程：使用相依資源建立 Azure Resource Manager 範本](./resource-manager-tutorial-create-templates-with-dependent-resources.md)。

## <a name="dependson"></a>dependsOn
在您的範本內，dependsOn 元素可讓您定義一個資源作為一或多個資源的相依項目。 其值可以是以逗號分隔的資源名稱清單。 

下列範例示範一個虛擬機器擴展集，此擴展集依存於負載平衡器、虛擬網路，以及會建立多個儲存體帳戶的迴圈。 這些其他資源不會顯示在下列範例中，但是必須存在於範本中其他的位置。

```json
{
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  "name": "[variables('namingInfix')]",
  "location": "[variables('location')]",
  "apiVersion": "2016-03-30",
  "tags": {
    "displayName": "VMScaleSet"
  },
  "dependsOn": [
    "[variables('loadBalancerName')]",
    "[variables('virtualNetworkName')]",
    "storageLoop",
  ],
  ...
}
```

在上述範例中，相依性是包含在透過名為 **storageLoop**複製迴圈所建立的資源上。 例如，請參閱 [在 Azure 資源管理員中建立多個資源的執行個體](resource-group-create-multiple.md)。

當定義相依性時，您可以包含資源提供者命名空間和資源類型，以避免模稜兩可。 比方說，若要釐清可能有和其他資源名稱相同的負載平衡器和虛擬網路，請使用下列格式：

```json
"dependsOn": [
  "[resourceId('Microsoft.Network/loadBalancers', variables('loadBalancerName'))]",
  "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]"
]
``` 

雖然您可能比較傾向於使用 dependsOn 來對應資源之間的關聯性，但是請務必了解為什麼您要這麼做。 例如，若要記載資源互連的方式，dependsOn 並不是適當的方法。 在部署之後，您便無法查詢 dependsOn 元素中定義了哪些資源。 使用 dependsOn 可能會影響部署時間，因為 Resource Manager 不會平行部署兩個具有相依性的資源。 

## <a name="child-resources"></a>子資源
resources 屬性可讓您指定與所定義的資源相關的子資源。 定義子資源時，深度只能有 5 層。 請務必注意，在子資源與父資源之間並不會建立隱含的相依性。 如果您需要在父資源之後部署子資源，您必須使用 dependsOn 屬性明確地敘述該相依性。 

每個父資源只接受特定的資源類型做為子資源。 可接受的資源類型是在父資源的 [範本結構描述](https://github.com/Azure/azure-resource-manager-schemas) 中指定。 子資源類型的名稱包含父資源類型的名稱，例如 **Microsoft.Web/sites/config** 和 **Microsoft.Web/sites/extensions** 兩者皆為 **Microsoft.Web/sites** 的子資源。

下列範例示範 SQL 伺服器和 SQL 資料庫。 請注意，SQL 資料庫與 SQL 伺服器之間定義明確相依性，即使資料庫是伺服器的子系也是一樣。

```json
"resources": [
  {
    "name": "[variables('sqlserverName')]",
    "type": "Microsoft.Sql/servers",
    "location": "[resourceGroup().location]",
    "tags": {
      "displayName": "SqlServer"
    },
    "apiVersion": "2014-04-01-preview",
    "properties": {
      "administratorLogin": "[parameters('administratorLogin')]",
      "administratorLoginPassword": "[parameters('administratorLoginPassword')]"
    },
    "resources": [
      {
        "name": "[parameters('databaseName')]",
        "type": "databases",
        "location": "[resourceGroup().location]",
        "tags": {
          "displayName": "Database"
        },
        "apiVersion": "2014-04-01-preview",
        "dependsOn": [
          "[variables('sqlserverName')]"
        ],
        "properties": {
          "edition": "[parameters('edition')]",
          "collation": "[parameters('collation')]",
          "maxSizeBytes": "[parameters('maxSizeBytes')]",
          "requestedServiceObjectiveName": "[parameters('requestedServiceObjectiveName')]"
        }
      }
    ]
  }
]
```

## <a name="reference-and-list-functions"></a>reference 和 list 函式
[reference 函式](resource-group-template-functions-resource.md#reference) 可讓運算式從其他 JSON 名稱和值組或執行階段資源衍生其值。 [list* 函式](resource-group-template-functions-resource.md#list)會從清單作業傳回資源的值。  當所參考的資源部署於同一個範本且依其名稱 (而非資源識別碼) 加以參考時，reference 和 list 運算式都會隱含地宣告某一個資源相依於另一個資源。 如果您將資源識別碼傳遞到 reference 或 list 函式，就不會建立隱含參考。

reference 函式的一般格式為：

```json
reference('resourceName').propertyPath
```

listKeys 函式的一般格式為：

```json
listKeys('resourceName', 'yyyy-mm-dd')
```

在下列範例中，CDN 端點明確相依於 CDN 設定檔，並隱含相依於 Web 應用程式。

```json
{
    "name": "[variables('endpointName')]",
    "type": "endpoints",
    "location": "[resourceGroup().location]",
    "apiVersion": "2016-04-02",
    "dependsOn": [
            "[variables('profileName')]"
    ],
    "properties": {
        "originHostHeader": "[reference(variables('webAppName')).hostNames[0]]",
        ...
    }
```

您可以使用此元素或 dependsOn 元素來指定相依性，但不需針對相同的相依資源使用這兩者。 請盡可能使用隱含的參考，以避免新增不必要的相依性。

若要深入了解，請參閱 [reference 函數](resource-group-template-functions-resource.md#reference)。

## <a name="recommendations-for-setting-dependencies"></a>設定相依性的建議

決定設定哪種相依性時，請使用下列指導方針︰

* 設定越少相依性越好。
* 設定子資源為相依於其父資源。
* 使用 **reference** 函式並傳入資源名稱，以針對必須共用屬性的資源，設定資源間的隱含相依性。 當您已經定義隱含的相依性時，不要新增明確的相依性 (**dependsOn**)。 這種方法可減少產生不必要相依性的風險。 
* 當資源必須有另一個資源的功能才能**建立**時，請設定相依性。 如果資源只會在部署之後互動，請勿設定相依性。
* 讓相依性重疊顯示，而不需要明確設定它們。 例如，您的虛擬機器相依於虛擬網路介面，而虛擬網路介面相依於虛擬網路和公用 IP 位址。 因此，會在所有三個資源之後部署虛擬機器，但不會明確地將虛擬機器設定為相依於所有三個資源。 這種方法可釐清相依性順序，而且稍後可以比較容易變更範本。
* 如果部署之前可以決定值，請嘗試部署沒有相依性的資源。 例如，如果設定值需要另一個資源的名稱，您可能就不需要相依性。 本指引並非一律有效，因為某些資源會確認其他資源是否存在。 如果您收到錯誤，請加入相依性。 

Resource Manager 範本會在驗證期間識別循環相依性。 如果收到錯誤表示循環相依性存在，請評估您的範本，以查看是否有任何不需要的相依性可以移除。 如果移除相依性沒有用，您可以將某些部署作業移到在有循環相依性的資源之後部署的子資源中，以避免循環相依性。 例如，假設您要部署兩部虛擬機器，但是您必須分別在上面設定互相參考的屬性。 您可以採取下列順序部署︰

1. vm1
2. vm2
3. vm1 的擴充相依於 vm1 和 vm2。 擴充在 vm1 上設定從 vm2 取得的值。
4. vm2 的擴充相依於 vm1 和 vm2。 擴充在 vm2 上設定從 vm1 取得的值。

如需評估部署順序及解決相依性錯誤的相關資訊，請參閱[使用 Azure Resource Manager 針對常見的 Azure 部署錯誤進行疑難排解](resource-manager-common-deployment-errors.md)。

## <a name="next-steps"></a>後續步驟

* 如須逐步瀏覽教學課程，請參閱[教學課程：使用相依資源建立 Azure Resource Manager 範本](./resource-manager-tutorial-create-templates-with-dependent-resources.md)。
* 若要了解在部署期間如何對相依性進行疑難排解，請參閱[使用 Azure Resource Manager 針對常見的 Azure 部署錯誤進行疑難排解](resource-manager-common-deployment-errors.md)。
* 若要了解如何建立 Azure 資源管理員範本，請參閱 [撰寫範本](resource-group-authoring-templates.md)。 
* 如需在範本中可用函式的清單，請參閱 [範本函式](resource-group-template-functions.md)。

