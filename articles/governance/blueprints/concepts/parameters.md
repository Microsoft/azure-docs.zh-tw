---
title: 在 Azure 藍圖中透過參數建立動態藍圖
description: 了解靜態與動態參數，以及如何使用它們來建立動態藍圖。
services: blueprints
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: conceptual
ms.service: blueprints
manager: carmonm
ms.openlocfilehash: e1a1d736eb9b22cd444a75b94d112abfe3fbe5af
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46993573"
---
# <a name="creating-dynamic-blueprints-through-parameters"></a>透過參數建立動態藍圖

使用各種成品 (例如，資源群組、Resource Manager 範本、原則或角色指派) 完整定義的藍圖，可以在 Azure 中快速建立並一致性地佈建物件。 為了能夠彈性使用這些可重複使用的設計模式與容器，Azure 藍圖支援參數。 參數會在定義與指派期間建立彈性，以變更藍圖所部署成品上的屬性。

資源群組成品即為一個簡單的範例。 建立資源群組時，必須提供兩個必要的值：名稱與位置。 將資源群組新增到藍圖時，如果參數不存在，則您需要在每次使用藍圖時定義該名稱與位置。 這導致每次使用藍圖時，就會在相同資源群組中建立成品。 儘管不會對資源群組本身造成任何問題，但該資源群組內的群組會變成重複且導致衝突。

> [!NOTE]
> 對於兩個不同的藍圖來說，包含相同名稱的資源群組並不會造成任何問題。
> 如果藍圖內含的資源群組已經存在，則藍圖會繼續在該資源群組中建立相關成品。 這可能導致衝突，因為具有相同名稱與資源類型的兩個資源不能存在於一個訂用帳戶內。

這是參數的適用情況。 那些屬性的值 (在資源群組的案例中為名稱與位置屬性)，藍圖讓您不需在藍圖定義期間定義它們，而是改為在指派至訂用帳戶期間定義其值。 這讓您能夠重複使用藍圖，在單一訂用帳戶中建立資源群組與其他資源，而不會發生衝突。

## <a name="blueprint-parameters"></a>藍圖參數

透過 REST API，除了每個支援的成品，還能在藍圖本身建立參數。 在藍圖上建立參數時，可透過該藍圖中的成品使用該參數。 用來為資源群組命名的前置詞即為範例之一。 成品接著可使用藍圖參數來建立「幾乎動態」的參數，因為該參數仍會在指派期間定義，但將具備可能繼承組織命名規則的一致性。 如需步驟，請參閱[設定靜態參數 - 藍圖層級參數](#blueprint-level-parameter)。

### <a name="using-securestring-and-secureobject-parameters"></a>使用 secureString 與 secureObject 參數

雖然 Resource Manager 範本「成品」支援 **secureString** 與 **secureObject** 類型的參數，Azure 藍圖要求每個都要與 Azure Key Vault 連線。
這可防止將祕密與藍圖儲存在一起的不安全作法，並鼓勵採用安全模式。 Azure 藍圖會透過偵測 Resource Manager 範本「成品」中包含安全參數，以及在針對每個偵測到的安全參數進行下列 Key Vault 屬性的藍圖指派期間進行提示，以簡化此作業：

- Key Vault 資源識別碼
- Key Vault 祕密名稱
- Key Vault 祕密版本

參考的 Key Vault 必須存在於相同的訂用帳戶中，因為藍圖已指派，而且也必須在 Key Vault 的 [存取原則] 頁面上設定 [為範本部署啟用對 Azure Resource Manager 的存取]。 如需如何啟用此功能的相關指示，請參閱 [Key Vault - 啟用範本部署](../../../managed-applications/key-vault-access.md#enable-template-deployment)。
如需 Azure Key Vault 的詳細資訊，請參閱 [Azure Key Vault 概觀](../../../key-vault/key-vault-overview.md)。

## <a name="parameter-types"></a>參數類型

### <a name="static-parameters"></a>靜態參數

藍圖定義中所定義的參數值稱為**靜態參數**。 這是因為每次使用藍圖時，都將使用該靜態值來部署成品。 在資源群組範例中，雖然這對資源群組名稱不具任何意義，但它對位置來說可能是有意義的。 接著，每個藍圖指派都會在相同的位置建立資源群組，而無論指派期間是否會呼叫它。 這讓您可以選擇要視需要定義哪些內容，以及可在指派期間變更哪些內容。

#### <a name="setting-static-parameters-in-the-portal"></a>在入口網站中設定靜態參數

1. 在左窗格中按一下 [所有服務]，然後搜尋並選取 [原則]，以在 Azure 入口網站中啟動 Azure 藍圖服務。 在 [原則] 頁面上，按一下 [藍圖]。

1. 在頁面左邊選取 [藍圖定義]。

1. 按一下現有的藍圖，然後按一下 [編輯藍圖]，或按一下 [+ 建立藍圖]，然後填寫 [基本] 索引標籤上的資訊。

1. 按一下 [下一步: 成品]，或按一下 [成品] 索引標籤。

1. 新增到具有參數選項之藍圖的成品會在 [參數] 欄中顯示 [已填入 X 個參數 (共 Y 個)]。 按一下成品列來編輯成品參數。

   ![藍圖參數](../media/parameters/parameter-column.png)

1. [編輯成品] 頁面會顯示適用於所按下之成品的值選項。 成品中的每個參數都有標題、值方塊與核取方塊。 將此方塊設定為未選取狀態，以使其成為**靜態參數**。 在下列範例中，只有「位置」是**靜態參數**，因為它已取消選取且 [資源群組名稱] 已選取。

   ![藍圖靜態參數](../media/parameters/static-parameter.png)

#### <a name="setting-static-parameters-from-rest-api"></a>從 REST API 設定靜態參數

在每個 REST API URI 中有一些變數，需要您以自己的值取代它們：

- `{YourMG}` - 以您的管理群組名稱取代
- `{subscriptionId}` - 以您的訂用帳戶識別碼取代

##### <a name="blueprint-level-parameter"></a>藍圖層級參數

透過 REST API 建立藍圖時，就能夠建立[藍圖參數](#blueprint-parameters)。 若要這樣做，請使用下列 REST API URI 與主體格式：

- REST API URI

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint?api-version=2017-11-11-preview
  ```

- 要求本文

  ```json
  {
      "properties": {
          "description": "This blueprint has blueprint level parameters.",
          "targetScope": "subscription",
          "parameters": {
              "owners": {
                  "type": "array",
                  "metadata": {
                      "description": "List of AAD object IDs that is assigned Owner role at the resource group"
                  }
              }
          },
          "resourceGroups": {
              "storageRG": {
                  "description": "Contains the resource template deployment and a role assignment."
              }
          }
      }
  }
  ```

建立藍圖層級參數之後，就能在已新增至該藍圖的成品中使用。
下列 REST API 範例會在藍圖上建立角色指派成品，並使用藍圖層級參數。

- REST API URI

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/roleOwner?api-version=2017-11-11-preview
  ```

- 要求本文

  ```json
  {
      "kind": "roleAssignment",
      "properties": {
          "resourceGroup": "storageRG",
          "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635",
          "principalIds": "[parameters('owners')]"
      }
  }
  ```

在此範例中，**principalIds** 屬性會透過提供 `[parameters('owners')]` 的值來使用**擁有者**藍圖層級參數。 使用藍圖層級參數在成品上設定參數，仍然是一個**靜態參數**範例，因為它無法在藍圖指派期間設定，而且在每個指派上都將是此值。

##### <a name="artifact-level-parameter"></a>成品層級參數

在成品上建立**靜態參數**的方式很類似，但會採用直接值，而非使用 `parameters()` 函式。 下列範例會建立兩個靜態參數，亦即 **tagName** 與 **tagValue**。 每個值都是直接提供的，不會使用函式呼叫。

- REST API URI

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/policyStorageTags?api-version=2017-11-11-preview
  ```

- 要求本文

  ```json
  {
      "kind": "policyAssignment",
      "properties": {
          "description": "Apply storage tag and the parameter also used by the template to resource groups",
          "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71",
          "parameters": {
              "tagName": {
                  "value": "StorageType"
              },
              "tagValue": {
                  "value": "Premium_LRS"
              }
          }
      }
  }
  ```

### <a name="dynamic-parameters"></a>動態參數

靜態參數的相反是**動態參數**。 這是是並未在藍圖中定義，而是改為在每個藍圖指派期間定義的參數。 在資源群組範例中，為每個藍圖指派提供不同名稱，對資源群組名稱來說很合理。

#### <a name="setting-dynamic-parameters-in-the-portal"></a>在入口網站中設定動態參數

1. 在左窗格中按一下 [所有服務]，然後搜尋並選取 [原則]，以在 Azure 入口網站中啟動 Azure 藍圖服務。 在 [原則] 頁面上，按一下 [藍圖]。

1. 在頁面左邊選取 [藍圖定義]。

1. 以滑鼠右鍵按一下您要指派的藍圖，然後選取 [指派藍圖]，或按一下您要指派的藍圖，然後按一下 [指派藍圖] 按鈕。

1. 在 [指派藍圖] 頁面上，尋找 [成品參數] 區段。 至少具有一個**動態參數**的每個成品都會顯示成品與設定選項。 指派藍圖之前，請先為參數提供所需的值。 在下列範例中，「名稱」是必須先定義才能完成藍圖指派的**動態參數**。

   ![藍圖動態參數](../media/parameters/dynamic-parameter.png)

#### <a name="setting-dynamic-parameters-from-rest-api"></a>從 REST API 設定動態參數

在指派期間設定**動態參數**，可透過直接提供所需的值來完成。 不使用函式 (例如 `parameters()`)，而是改為提供適當字串的值。 資源群組的成品是使用「範本名稱」與**名稱**與**位置**屬性來定義。 每個內含成品的所有其他參數都定義在**參數**之下，並具備 **\<名稱\>** 與**值**機碼值組。 如果已針對指派期間未提供的動態參數設定藍圖，指派將會失敗。

- REST API URI

  ```http
  PUT https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Blueprint/blueprintAssignments/assignMyBlueprint?api-version=2017-11-11-preview
  ```

- 要求本文

  ```json
  {
      "properties": {
          "blueprintId": "/providers/Microsoft.Management/managementGroups/{YourMG}  /providers/Microsoft.Blueprint/blueprints/MyBlueprint",
          "resourceGroups": {
              "storageRG": {
                  "name": "StorageAccount",
                  "location": "eastus2"
              }
          },
          "parameters": {
              "storageAccountType": {
                  "value": "Standard_GRS"
              },
              "tagName": {
                  "value": "CostCenter"
              },
              "tagValue": {
                  "value": "ContosoIT"
              },
              "contributors": {
                  "value": [
                      "7be2f100-3af5-4c15-bcb7-27ee43784a1f",
                      "38833b56-194d-420b-90ce-cff578296714"
                  ]
                },
              "owners": {
                  "value": [
                      "44254d2b-a0c7-405f-959c-f829ee31c2e7",
                      "316deb5f-7187-4512-9dd4-21e7798b0ef9"
                  ]
              }
          }
      },
      "identity": {
          "type": "systemAssigned"
      },
      "location": "westus"
  }
  ```

## <a name="next-steps"></a>後續步驟

- 了解[藍圖生命週期](lifecycle.md)
- 了解如何自訂[藍圖排序順序](sequencing-order.md)
- 了解如何使用[藍圖資源鎖定](resource-locking.md)
- 了解如何[更新現有的指派](../how-to/update-existing-assignments.md)
- 使用[一般疑難排解](../troubleshoot/general.md)來解決藍圖指派期間發生的問題