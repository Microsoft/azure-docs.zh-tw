---
title: Azure API 管理中的原則 | Microsoft Docs
description: 了解如何在 API 管理中建立、編輯和設定原則。
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
ms.date: 11/29/2017
ms.author: apimpm
ms.openlocfilehash: 54fbba197f6609731ffaf3ff15143a28e70a955f
ms.sourcegitcommit: 83ea7c4e12fc47b83978a1e9391f8bb808b41f97
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/28/2018
ms.locfileid: "29712863"
---
# <a name="policies-in-azure-api-management"></a>Azure API 管理中的原則

在 Azure API 管理 (APIM) 中，原則是系統的一項強大功能，可讓發行者透過組態來變更 API 的行為。 原則是「陳述式」的集合，會在 API 的要求或回應上循序執行。 常見的「陳述式」包括從 XML 至 JSON 的格式轉換，以及利用呼叫速率限制來限制開發人員傳入的呼叫數量。 還有許多現成的原則可用。

原則是在位於 API 取用者與受控 API 之間的閘道內套用。 閘道會接收所有要求，然後通常原封不動地轉送至基礎 API。 不過，原則可以套用至輸入要求和輸出要求。

如果原則不另行指定，則可以在任何 API 管理原則中，使用原則運算式做為屬性值或文字值。 某些原則是以原則運算式為基礎，例如[控制流程][Control flow]和[設定變數][Set variable]原則。 如需詳細資訊，請參閱[進階原則][Advanced policies]和[原則運算式][Policy expressions]。

## <a name="sections"> </a>了解原則組態

原則定義是一份簡單的 XML 文件，描述一連串輸入和輸出陳述式。 可直接在定義視窗中編輯 XML。 右邊提供陳述式的清單，而且會啟用並醒目提示適用於目前範圍的陳述式。

按一下已啟用的陳述式會在定義檢視中的游標位置上加入適當的 XML。 

> [!NOTE]
> 如果未啟用您想要新增的原則，請確定您是在該原則的正確範圍內。 每個原則陳述式都是針對在特定範圍和原則區段中使用所設計。 若要檢閱原則的原則區段和範圍，請檢查[原則參考][Policy Reference]中該原則的**使用方式**一節。
> 
> 

設定分為 `inbound`、`backend`、`outbound` 和 `on-error`。 指定的一連串原則陳述式會針對要求和回應而依序執行。

```xml
<policies>
  <inbound>
    <!-- statements to be applied to the request go here -->
  </inbound>
  <backend>
    <!-- statements to be applied before the request is forwarded to 
         the backend service go here -->
  </backend>
  <outbound>
    <!-- statements to be applied to the response go here -->
  </outbound>
  <on-error>
    <!-- statements to be applied if there is an error condition go here -->
  </on-error>
</policies> 
```

若在處理要求期間發生錯誤，便會略過 `inbound`、`backend` 或 `outbound` 區段中的所有剩餘步驟，且執行程序會跳至 `on-error` 區段中的陳述式。 將原則陳述式置於 `on-error` 區段中，您即可使用 `context.LastError` 屬性檢閱錯誤、使用 `set-body` 原則檢查和自訂錯誤回應，以及設定錯誤發生時採取的動作。 會出現內建步驟的錯誤碼和處理原則陳述式期間可能會發生之錯誤的錯誤碼。 如需詳細資訊，請參閱 [API 管理原則中的錯誤處理方式](https://msdn.microsoft.com/library/azure/mt629506.aspx)。

## <a name="scopes"> </a>如何設定原則

如需如何設定原則的資訊，請參閱[設定或編輯原則](set-edit-policies.md)。

## <a name="policy-reference"></a>原則參考文件

如需原則陳述式及其設定的完整清單，請參閱[原則參考文件](api-management-policy-reference.md)。

## <a name="policy-samples"></a>原則範例

請參閱[原則範例](policy-samples.md)以取得更多程式碼範例。

## <a name="examples"></a>範例

### <a name="apply-policies-specified-at-different-scopes"></a>套用在不同範圍指定的原則

若您在全域層級中有一個原則，還有一個為 API 設定的原則，則每次使用該特定 API 時，皆會套用這兩個原則。 API 管理可透過 base 元素來指定組合式原則陳述式的固定順序。 

```xml
<policies>
    <inbound>
        <cross-domain />
        <base />
        <find-and-replace from="xyz" to="abc" />
    </inbound>
</policies>
```

在上述的原則定義範例中，`cross-domain` 陳述式會在任何更高層級的原則執行之前執行，而這些原則後面又接著 `find-and-replace` 原則。 

### <a name="restrict-incoming-requests"></a>限制傳入要求

若要新增陳述式以限制接收指定 IP 位址的傳入要求，請將游標移至 `inbound` XML 元素的內容當中，然後按一下 **Restrict caller IPs** 陳述式。

![Restriction policies][policies-restrict]

這會將 XML 程式碼片段新增至提供設定陳述式之指引的 `inbound` 元素。

```xml
<ip-filter action="allow | forbid">
    <address>address</address>
    <address-range from="address" to="address"/>
</ip-filter>
```

若要限制輸入要求，只接受來自 IP 位址 1.2.3.4 的要求，請如下修改 XML：

```xml
<ip-filter action="allow">
    <address>1.2.3.4</address>
</ip-filter>
```

## <a name="next-steps"></a>後續步驟

如需使用原則的詳細資訊，請參閱︰

+ [轉換 API](transform-api.md)
+ [原則參考文件](api-management-policy-reference.md)，取得原則陳述式及其設定的完整清單
+ [原則範例](policy-samples.md)   

[Policy Reference]: api-management-policy-reference.md
[Product]: api-management-howto-add-products.md
[API]: api-management-howto-add-products.md
[Operation]: api-management-howto-add-operations.md

[Advanced policies]: https://msdn.microsoft.com/library/azure/dn894085.aspx
[Control flow]: https://msdn.microsoft.com/library/azure/dn894085.aspx#choose
[Set variable]: https://msdn.microsoft.com/library/azure/dn894085.aspx#set_variable
[Policy expressions]: https://msdn.microsoft.com/library/azure/dn910913.aspx

[policies-restrict]: ./media/api-management-howto-policies/api-management-policies-restrict.png
