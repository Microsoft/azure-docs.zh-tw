---
title: 如何在 Azure API 管理原則中使用具名值
description: 了解如何在 Azure API 管理原則中使用具名值。
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/25/2018
ms.author: apimpm
ms.openlocfilehash: 829d6bc6cb3f8e78d065d7aaca4937634e7349c8
ms.sourcegitcommit: e0834ad0bad38f4fb007053a472bde918d69f6cb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37437060"
---
# <a name="how-to-use-named-values-in-azure-api-management-policies"></a>如何在 Azure API 管理原則中使用具名值
API 管理原則是系統的強大功能，可讓 Azure 入口網站透過設定來變更 API 的行為。 原則是陳述式的集合，會因 API 的要求或回應循序執行。 原則陳述式可以使用常值文字值、原則運算式和具名值來建構。 

每個 API 管理服務執行個體都有服務執行個體全域適用的的鍵/值組 (稱為具名值) 的屬性集合。 這個具名值可用來管理所有 API 組態及原則的常數字串值。 每個屬性可能都有下列屬性：

| 屬性 | 類型 | 說明 |
| --- | --- | --- |
| 顯示名稱 |字串 |用於參考原則中屬性的英數字元字串。 |
| 值 |字串 |屬性的值。 不能是空白或只由空白字元組成。 |
|Secret|布林值|決定該值是否為密碼且是否應該加密。|
| 標記 |字串陣列 |若有提供選用的標記，則可用來篩選屬性清單。 |

![具名值](./media/api-management-howto-properties/named-values.png)

屬性值可以包含常值字串及 [原則運算式](https://msdn.microsoft.com/library/azure/dn910913.aspx)。 例如，`ExpressionProperty` 的值是原則運算式，會傳回包含目前日期與時間的字串。 `ContosoHeaderValue` 屬性已標記為密碼，所以未顯示其值。

| Name | 值 | Secret | 標記 |
| --- | --- | --- | --- |
| ContosoHeader |TrackingId |False |Contoso |
| ContosoHeaderValue |•••••••••••••••••••••• |True |Contoso |
| ExpressionProperty |@(DateTime.Now.ToString()) |False | |

## <a name="to-add-and-edit-a-property"></a>若要新增及編輯屬性

![新增屬性](./media/api-management-howto-properties/add-property.png)

1. 選取 [API 管理] 下方的 [API]。
2. 選取 [具名值]。
3. 按 [+新增]。

  [名稱] 和 [值] 都是必要值。 如果此屬性值是祕密，請勾選 [這是祕密] 核取方塊。 輸入一或多個選擇性標籤來協助組織您的具名值，然後按一下 [儲存]。
4. 按一下頁面底部的 [新增] 。

屬性建立之後，您可以按一下屬性來加以編輯。 如果您變更屬性名稱，任何參照該屬性的原則會自動更新以使用新的名稱。

如需使用 REST API 編輯屬性的詳細資訊，請參閱 [使用 REST API 編輯屬性](https://msdn.microsoft.com/library/azure/mt651775.aspx#Patch)。

## <a name="to-delete-a-property"></a>刪除屬性

若要刪除屬性，請按一下屬性旁邊的 [刪除]  來刪除。

> [!IMPORTANT]
> 如有任何原則參照該屬性，則您必須將該屬性從所有使用它的原則中移除，才能成功刪除該屬性。
> 
> 

如需使用 REST API 刪除屬性的詳細資訊，請參閱 [使用 REST API 刪除屬性](https://msdn.microsoft.com/library/azure/mt651775.aspx#Delete)。

## <a name="to-search-and-filter-named-values"></a>搜尋與篩選具名值

[具名值] 索引標籤包括可協助您管理具名值的搜尋與篩選功能。 若要按照屬性名稱篩選屬性清單，請在 [搜尋屬性]  文字方塊中輸入搜尋字詞。 若要顯示所有具名值，請清除 [搜尋屬性] 文字方塊，然後按 Enter 鍵。

若要按照標籤值篩選屬性清單，請在 [依標籤篩選]  文字方塊中輸入一或多個標籤。 若要顯示所有具名值，請清除 [依標記篩選] 文字方塊，然後按 Enter 鍵。

## <a name="to-use-a-property"></a>使用屬性

若要在原則中使用屬性，請將屬性名稱放在雙大括號 (如 `{{ContosoHeader}}`) 內，如以下範例所示：

```xml
<set-header name="{{ContosoHeader}}" exists-action="override">
  <value>{{ContosoHeaderValue}}</value>
</set-header>
```

在此範例中，`ContosoHeader` 是做為 `set-header` 原則中標頭的名稱，且 `ContosoHeaderValue` 是用來做為該標頭的值。 當此原則在對 API 管理閘道提出要求或回應管理閘道期間被評估時，`{{ContosoHeader}}` 和 `{{ContosoHeaderValue}}` 會被其各自的屬性值取代。

如前述範例所示，具名值可以作為完整的屬性或元素值使用，但它們也可以插入部分的常值文字運算式或與之結合，如以下範例所示：`<set-header name = "CustomHeader{{ContosoHeader}}" ...>`

具名值也可以包含原則運算式。 以下範例使用 `ExpressionProperty`。

```xml
<set-header name="CustomHeader" exists-action="override">
    <value>{{ExpressionProperty}}</value>
</set-header>
```

當評估此原則時，`{{ExpressionProperty}}` 會替代為其值 `@(DateTime.Now.ToString())`。 因為該值是原則運算式，所以會評估運算式，且原則會繼續執行。

您可以在開發人員入口網站測試此項目，方法是呼叫原則中包含範圍內具名值的作業。 在下列範例中，會使用前兩個含具名值的範例 `set-header` 原則來呼叫作業。 請注意，回應中包含兩個使用含具名值的原則設定的自訂標頭。

![開發人員入口網站][api-management-send-results]

如果您在 [API 檢查器追蹤](api-management-howto-api-inspector.md)中查看包含前述兩個含具名值之範例原則的呼叫，您會看到兩個已插入屬性值的 `set-header` 原則，以及包含原則運算式之屬性的原則運算式評估。

![API 檢查器追蹤][api-management-api-inspector-trace]

雖然屬性值可以包含原則運算式，但屬性值不可包含其他具名值。 如果文字包含做為屬性值的屬性參照 (例如 `Property value text {{MyProperty}}`)，該屬性參照不會被取代，且將會包含做為屬性值的一部分。

## <a name="next-steps"></a>後續步驟
* 深入了解原則的使用方式
  * [API 管理中的原則](api-management-howto-policies.md)
  * [原則參考文件](https://msdn.microsoft.com/library/azure/dn894081.aspx)
  * [原則運算式](https://msdn.microsoft.com/library/azure/dn910913.aspx)

[api-management-send-results]: ./media/api-management-howto-properties/api-management-send-results.png
[api-management-properties-filter]: ./media/api-management-howto-properties/api-management-properties-filter.png
[api-management-api-inspector-trace]: ./media/api-management-howto-properties/api-management-api-inspector-trace.png

