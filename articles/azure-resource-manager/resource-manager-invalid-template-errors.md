---
title: Azure 無效範本錯誤 | Microsoft Docs
description: 描述如何解決無效範本錯誤。
services: azure-resource-manager
documentationcenter: ''
author: tfitzmac
manager: timlt
editor: ''
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 03/08/2018
ms.author: tomfitz
ms.openlocfilehash: 59f07b9ba8116cb1a4b5ab50382d89d01a78853b
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34357685"
---
# <a name="resolve-errors-for-invalid-template"></a>解決無效範本的錯誤

本文描述如何解決無效範本錯誤。

## <a name="symptom"></a>徵狀

在部署範本時您收到錯誤，指出：

```
Code=InvalidTemplate
Message=<varies>
```

錯誤訊息取決於錯誤類型。

## <a name="cause"></a>原因

此錯誤可能起因於數個不同類型的錯誤。 通常涉及範本中的語法或結構錯誤。

<a id="syntax-error" />

## <a name="solution-1---syntax-error"></a>解決方案 1 - 語法錯誤

如果您收到錯誤訊息指出無法驗證範本，則可能是範本中發生語法問題。

```
Code=InvalidTemplate
Message=Deployment template validation failed
```

此錯誤很容易發生，因為範本運算式可能很複雜。 例如，儲存體帳戶的下列名稱指派含有一組方括號、三個函式、三組圓括號、一組單引號和一個屬性：

```json
"name": "[concat('storage', uniqueString(resourceGroup().id))]",
```

如果您未提供相符的語法，範本會產生與您所要的值截然不同的值。

當您收到此類錯誤時，請仔細檢閱運算式語法。 請考慮使用 [Visual Studio](vs-azure-tools-resource-groups-deployment-projects-create-deploy.md) 或 [Visual Studio Code](resource-manager-vs-code.md) 等 JSON 編輯器，它們可以警告您有語法錯誤。

<a id="incorrect-segment-lengths" />

## <a name="solution-2---incorrect-segment-lengths"></a>解決方案 2 - 不正確的區段長度

資源名稱的格式不正確時，就會發生另一種無效範本錯誤。

```
Code=InvalidTemplate
Message=Deployment template validation failed: 'The template resource {resource-name}'
for type {resource-type} has incorrect segment lengths.
```

根層級資源的名稱必須比資源類型少一個區段。 每個區段是以斜線區分。 在下列範例中，類型有兩個區段，而名稱有一個區段，因此它是**有效名稱**。

```json
{
  "type": "Microsoft.Web/serverfarms",
  "name": "myHostingPlanName",
  ...
}
```

但下一個範例則 **不是有效名稱** ，因為它的區段數目和類型相同。

```json
{
  "type": "Microsoft.Web/serverfarms",
  "name": "appPlan/myHostingPlanName",
  ...
}
```

對於子資源來說，類型和名稱具有相同的區段數目。 此區段數目很合理，因為子資源的完整名稱和類型包含父資源的名稱和類型。 因此，完整名稱較完整類型仍少一個區段。

```json
"resources": [
    {
        "type": "Microsoft.KeyVault/vaults",
        "name": "contosokeyvault",
        ...
        "resources": [
            {
                "type": "secrets",
                "name": "appPassword",
                ...
            }
        ]
    }
]
```

對於跨資源提供者套用的 Resource Manager 類型而言，要讓區段保持正確可能很棘手。 例如，將資源鎖定套用至網站需要具有四個區段的類型。 因此，名稱會是三個區段：

```json
{
    "type": "Microsoft.Web/sites/providers/locks",
    "name": "[concat(variables('siteName'),'/Microsoft.Authorization/MySiteLock')]",
    ...
}
```

<a id="parameter-not-valid" />

## <a name="solution-3---parameter-is-not-valid"></a>解決方案 3 - 參數無效

如果您提供的參數值不是其中一個允許值，則會收到類似下列錯誤的訊息：

```
Code=InvalidTemplate;
Message=Deployment template validation failed: 'The provided value {parameter value}
for the template parameter {parameter name} is not valid. The parameter value is not
part of the allowed values
```

在範本中重複檢查允許的值，並在部署期間提供一個值。 如需允許的參數值詳細資訊，請參閱 [Azure Resource Manager 範本的 Parameters 區段](resource-manager-templates-parameters.md)。

<a id="too-many-resource-groups" />

## <a name="solution-4---too-many-target-resource-groups"></a>解決方案 4 - 太多目標資源群組

如果您在單一部署中指定五個以上的目標資源群組，就會收到此錯誤。 請考慮合併部署中的資源群組數目，或將一些範本部署為個別部署。 如需詳細資訊，請參閱[將 Azure 資源部署至多個訂用帳戶和資源群組](resource-manager-cross-resource-group-deployment.md)。

<a id="circular-dependency" />

## <a name="solution-5---circular-dependency-detected"></a>解決方案 5 - 偵測到循環相依性

當資源以防止部署啟動的方式互相相依，您會收到這個錯誤。 只要有相互相依性的組合，就會讓兩個或多個資源等候其他也正在等候的資源。 例如，resource1 相依於 resource3、resource2 相依於 resource1，而 resource3 相依於 resource2。 您通常可以移除不必要的相依性來解決此問題。

解決循環相依性︰

1. 在範本中，找出循環相依性中所識別的資源。 
2. 針對該資源，檢查 **dependsOn** 屬性和任何使用的 **reference** 函式，查看相依於哪些資源。 
3. 檢查這些資源，查看所相依的資源。 跟隨相依性，直到您找出相依於原始資源的資源。
5. 針對參與循環相依性的資源，仔細檢查所有使用的 **dependsOn** 屬性，以識別不需要的任何相依性。 移除這些相依性。 如果您不確定是否需要某個相依性，請嘗試移除它。 
6. 重新部署範本。

移除 **dependsOn** 屬性的值可能會在您部署範本時導致錯誤。 如果您收到錯誤，請將相依性新增回範本。 

如果該方法無法解決循環相依性，請考慮將一部分部署邏輯移到子資源 (例如擴充功能或設定值)。 設定這些子資源在參與循環相依性的資源之後才進行部署。 例如，假設您要部署兩部虛擬機器，但是您必須分別在上面設定互相參考的屬性。 您可以採取下列順序部署︰

1. vm1
2. vm2
3. vm1 的擴充相依於 vm1 和 vm2。 擴充在 vm1 上設定從 vm2 取得的值。
4. vm2 的擴充相依於 vm1 和 vm2。 擴充在 vm2 上設定從 vm1 取得的值。

相同的方法也適用於 App Service 應用程式。 請考慮將設定值移入應用程式資源的子資源。 您可以採取下列順序部署兩個 Web 應用程式︰

1. webapp1
2. webapp2
3. webapp1 的設定相依於 webapp1 和 webapp2。 它包含使用 webapp2 的值的應用程式設定。
4. webapp2 的設定相依於 webapp1 和 webapp2。 它包含使用 webapp1 的值的應用程式設定。
