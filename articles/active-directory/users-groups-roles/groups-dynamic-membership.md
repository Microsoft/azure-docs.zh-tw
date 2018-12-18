---
title: Azure Active Directory 中的自動群組成員資格規則參考 | Microsoft Docs
description: 如何建立成員資格規則，以自動填入群組和規則參考。
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.component: users-groups-roles
ms.topic: article
ms.date: 09/20/2018
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.openlocfilehash: c3121f8b303d9f82ed949d598a942906d0d24f7e
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47041018"
---
# <a name="dynamic-membership-rules-for-groups-in-azure-active-directory"></a>Azure Active Directory 中群組的動態成員資格規則

在 Azure Active Directory (Azure AD) 中，您可以建立以複雜屬性為基礎的規則，來啟用群組的動態成員資格。 動態群組成員資格可減少新增及移除使用者的額外管理負擔。 本文將詳細說明用以建立適用於使用者或裝置之動態成員資格規則的屬性和語法。 您可以為安全性群組或 Office 365 群組的動態成員資格設定規則。

當使用者或裝置的任何屬性變更時，系統會評估目錄中的所有動態群組規則，以查看變更是否會觸發任何的群組新增或移除。 如果使用者或裝置滿足群組上的規則，就會將他們新增為該群組的成員。 如果他們不再符合此規則，則會予以移除。 您無法手動新增或移除動態群組的成員。

* 您可以為裝置或使用者建立動態群組，但無法建立同時包含使用者和裝置的規則。
* 您無法根據裝置擁有者的屬性來建立裝置群組。 裝置成員資格規則只能參考裝置屬性。

> [!NOTE]
> 此功能需要一或多個動態群組成員中每個唯一使用者的 Azure AD Premium P1 授權。 您不需要將授權指派給使用者，使用者就能成為動態群組成員，但是您必須在租用戶中有最小數量的授權，才能涵蓋所有這類使用者。 例如，如果您租用戶中的所有動態群組總計有 1,000 位唯一使用者，則需要至少有 Azure AD Premium P1 的 1,000 個授權，才符合授權需求。
>

## <a name="constructing-the-body-of-a-membership-rule"></a>建構成員資格規則的本文

自動填入使用者或裝置群組的成員資格規則是可導致 true 或 false 結果的二進位運算式。 簡單規則的三個部分如下：

* 屬性
* 運算子
* 值

運算式內的部分順序很重要，可避免發生語法錯誤。

### <a name="rules-with-a-single-expression"></a>使用單一運算式的規則

單一運算式是最簡單的成員資格規則形式，只包含上述三個部分。 具有單一運算式的規則看起來像這樣：`Property Operator Value`，其中屬性的語法是 object.property 的名稱。

以下範例是正確建構的成員資格規則 (具有單一運算式)：

```
user.department -eq "Sales"
```

單一運算式的括號是選擇性的。 成員資格規則本文的總長度不得超過 2048 個字元。

## <a name="supported-properties"></a>支援的屬性

有三種類型的屬性可用來建構成員資格規則。

* BOOLEAN
* 字串
* 字串集合

以下是您可用來建立單一運算式的使用者屬性。

### <a name="properties-of-type-boolean"></a>布林型別的屬性

| properties | 允許的值 | 使用量 |
| --- | --- | --- |
| accountEnabled |true false |user.accountEnabled -eq true |
| dirSyncEnabled |true false |user.dirSyncEnabled -eq true |

### <a name="properties-of-type-string"></a>字串類型的屬性

| properties | 允許的值 | 使用量 |
| --- | --- | --- |
| city |任何字串值或 *null* |(user.city -eq "value") |
| country |任何字串值或 *null* |(user.country -eq "value") |
| companyName | 任何字串值或 *null* | (user.companyName -eq "value") |
| department |任何字串值或 *null* |(user.department -eq "value") |
| displayName |任何字串值 |(user.displayName -eq "value") |
| employeeId |任何字串值 |(user.employeeId -eq "value")<br>(user.employeeId -ne *null*) |
| facsimileTelephoneNumber |任何字串值或 *null* |(user.facsimileTelephoneNumber -eq "value") |
| givenName |任何字串值或 *null* |(user.givenName -eq "value") |
| jobTitle |任何字串值或 *null* |(user.jobTitle -eq "value") |
| mail |任何字串值或 *null* (使用者的 SMTP 位址) |(user.mail -eq "value") |
| mailNickName |任何字串值 (使用者的郵件別名) |(user.mailNickName -eq "value") |
| mobile |任何字串值或 *null* |(user.mobile -eq "value") |
| objectId |使用者物件的 GUID |(user.objectId -eq "11111111-1111-1111-1111-111111111111") |
| onPremisesSecurityIdentifier | 已從內部部署環境同步至雲端之使用者的內部部署安全性識別碼 (SID)。 |(user.onPremisesSecurityIdentifier -eq "S-1-1-11-1111111111-1111111111-1111111111-1111111") |
| passwordPolicies |None DisableStrongPassword DisablePasswordExpiration DisablePasswordExpiration, DisableStrongPassword |(user.passwordPolicies -eq "DisableStrongPassword") |
| physicalDeliveryOfficeName |任何字串值或 *null* |(user.physicalDeliveryOfficeName -eq "value") |
| postalCode |任何字串值或 *null* |(user.postalCode -eq "value") |
| preferredLanguage |ISO 639-1 code |(user.preferredLanguage -eq "en-US") |
| sipProxyAddress |任何字串值或 *null* |(user.sipProxyAddress -eq "value") |
| state |任何字串值或 *null* |(user.state -eq "value") |
| streetAddress |任何字串值或 *null* |(user.streetAddress -eq "value") |
| surname |任何字串值或 *null* |(user.surname -eq "value") |
| telephoneNumber |任何字串值或 *null* |(user.telephoneNumber -eq "value") |
| usageLocation |兩個字母的國家 (地區) 代碼 |(user.usageLocation -eq "US") |
| userPrincipalName |任何字串值 |(user.userPrincipalName -eq "alias@domain") |
| userType |member guest *null* |(user.userType -eq "Member") |

### <a name="properties-of-type-string-collection"></a>字串集合類型的屬性

| properties | 允許的值 | 使用量 |
| --- | --- | --- |
| otherMails |任何字串值 |(user.otherMails -contains "alias@domain") |
| proxyAddresses |SMTP: alias@domain smtp: alias@domain |(user.proxyAddresses -contains "SMTP: alias@domain") |

如需裝置規則所使用的屬性，請參閱[裝置的規則](#rules-for-devices)。

## <a name="supported-operators"></a>支援的運算子

下表列出單一運算式支援的所有運算子及其語法。 不管有沒有連字號 (-) 前置詞，均可使用運算子。

| 運算子 | 語法 |
| --- | --- |
| Not Equals |-ne |
| Equals |-eq |
| Not Starts With |-notStartsWith |
| 開頭為 |-startsWith |
| Not Contains |-notContains |
| Contains |-contains |
| Not Match |-notMatch |
| Match |-match |
| 在 | -in |
| 不在 | -notIn |

### <a name="using-the--in-and--notin-operators"></a>使用 -In 和 -notIn 運算子

如果您想要針對許多不同的值比較使用者屬性的值，您可以使用 -In 或 -notIn 運算子。 使用方括號 "[" 和 "]" 作為值清單的開頭和結尾。

 在以下範例中，如果 user.department 的值等於清單中的任何一個值，此運算式會評估為 True：

```
   user.department -In ["50001","50002","50003",“50005”,“50006”,“50007”,“50008”,“50016”,“50020”,“50024”,“50038”,“50039”,“51100”]
```

## <a name="supported-values"></a>支援的值

運算式中使用的值可以包含數種類型，包括：

* 字串
* 布林值 – true、false
* 數字
* 陣列 – 數字陣列、字串陣列

指定運算式中的值時，務必使用正確的語法，以避免發生錯誤。 一些語法秘訣如下：

* 除非值為字串，否則雙引號是選擇性的。
* 字串和 regex 運算都不區分大小寫。
* 當字串值包含雙引號時，這兩個引號應該會使用 \` 字元逸出，例如 user.department -eq \`"Sales\`" 是值為 "Sales" 時的正確語法。
* 您也可以使用 null 值來執行 Null 檢查，例如 `user.department -eq null`。

### <a name="use-of-null-values"></a>使用 Null 值

若要在規則中指定 null 值，您可以使用 *null* 值。 

* 在比較運算式中的 *null* 值時，請使用 -eq 或 -ne。
* 只有在想要將 *null* 一字解釋為常值字串值時，才在它的前後使用引號。
* -not 運算子不能用來作為 null 的比較運算子。 如果使用它，則不論您使用 null 還是 $null 都會收到錯誤。

參考 null 值的正確方法如下：

```
   user.mail –ne null
```

## <a name="rules-with-multiple-expressions"></a>具有多個運算式的規則

群組成員資格規則可以包含多個以 -and、 -or 和 -not 邏輯運算子連接的運算式。 邏輯運算子也可以合併使用。 

以下範例是正確建構的成員資格規則 (具有多個運算式)：

```
(user.department -eq "Sales") -or (user.department -eq "Marketing")
(user.department -eq "Sales") -and -not (user.jobTitle -contains "SDE")
```

### <a name="operator-precedence"></a>運算子優先順序

所有運算子都會列在下面 (由高至低排定優先順序)。 同一行上運算子的優先順序相等：

```
-eq -ne -startsWith -notStartsWith -contains -notContains -match –notMatch -in -notIn
-not
-and
-or
-any -all
```

以下是運算子優先順序範例，其中有兩個運算式會針對使用者而評估：

```
   user.department –eq "Marketing" –and user.country –eq "US"
```

優先順序不符合您的需求時，才需要括號。 例如，如果您想要先評估部門，以下顯示如何使用括號來決定順序：

```
   user.country –eq "US" –and (user.department –eq "Marketing" –or user.department –eq "Sales")
```

## <a name="rules-with-complex-expressions"></a>具有複雜運算式的規則

成員資格規則可由複雜的運算式所組成，其中的屬性、運算子和值是在更複雜的表單上取得。 當下列任一項成立時，即可視為複雜運算式：

* 屬性由值集合所組成；具體來說，就是多重值的屬性
* 運算式會使用 -any 和 -all 運算子
* 運算式的值本身可以是一或多個運算式

## <a name="multi-value-properties"></a>多重值屬性

多重值屬性是相同類型之物件的集合。 它們可以用來建立使用 -any 和 -all 邏輯運算子的成員資格規則。

| properties | 值 | 使用量 |
| --- | --- | --- |
| assignedPlans | 集合中的每個物件都會公開下列字串屬性：capabilityStatus、service、servicePlanId |user.assignedPlans -any (assignedPlan.servicePlanId -eq "efb87545-963c-4e0d-99df-69c6916d9eb0" -and assignedPlan.capabilityStatus -eq "Enabled") |
| proxyAddresses| SMTP: alias@domain smtp: alias@domain | (user.proxyAddresses -any (\_ -contains "contoso")) |

### <a name="using-the--any-and--all-operators"></a>使用 -any 和 -all 運算子

您可以使用 -any 和 -all 運算子，分別將條件套用至集合中的一個或所有項目。

* -any (集合中至少有一個項目符合條件時滿足)
* -all (集合中的所有項目符合條件時滿足)

#### <a name="example-1"></a>範例 1

assignedPlans 是多重值屬性，可列出所有指派給使用者的服務方案。 下列運算式會選取擁有 Exchange Online (方案 2) 服務方案 (作為 GUID 值) 的使用者 (該方案也處於 [已啟用] 狀態)：

```
user.assignedPlans -any (assignedPlan.servicePlanId -eq "efb87545-963c-4e0d-99df-69c6916d9eb0" -and assignedPlan.capabilityStatus -eq "Enabled")
```

這類規則可用來將所有已啟用 Office 365 (或其他 Microsoft 線上服務) 功能的使用者分組。 您可以接著將一組原則套用到群組。

#### <a name="example-2"></a>範例 2

下列運算式會選取有任何服務方案與 Intune 服務 (透過服務名稱 "SCO" 來識別) 相關聯的所有使用者：

```
user.assignedPlans -any (assignedPlan.service -eq "SCO" -and assignedPlan.capabilityStatus -eq "Enabled")
```

### <a name="using-the-underscore--syntax"></a>使用底線 (\_) 語法

底線 (\_) 語法會比對發生在其中一個多重值字串集合屬性中的特定值，以將使用者或裝置新增至動態群組。 會搭配使用 -any 或 -all 運算子。

以下範例是在規則中使用底線 (\_)，根據 user.proxyAddress (與 user.otherMails 有相同的效果) 來新增成員。 此規則會將 Proxy 位址中包含 "contoso" 的任何使用者新增至群組。

```
(user.proxyAddresses -any (_ -contains "contoso"))
```

## <a name="other-properties-and-common-rules"></a>其他屬性和通用規則

### <a name="create-a-direct-reports-rule"></a>建立「直屬員工」規則

您可以建立一個群組，其中包含某位經理的所有直屬員工。 當經理的直屬員工在未來變更時，系統便會自動調整群組的成員資格。

使用下列語法來建構直屬員工規則：

```
Direct Reports for "{objectID_of_manager}"
```

以下是有效的規則範例，其中 “62e19b97-8b3d-4d4a-a106-4ce66896a863” 為管理員的 objectID：

```
Direct Reports for "62e19b97-8b3d-4d4a-a106-4ce66896a863"
```

下列秘訣可協助您正確使用此規則。

* [經理識別碼]是經理的物件識別碼。 在經理的**設定檔**可找到它。
* 若要讓規則得以運作，請確定已對您租用戶中的使用者正確設定 [經理] 屬性。 您可以在使用者的 [設定檔] 中檢查目前值。
* 此規則僅支援經理的直屬員工。 換句話說，您建立的群組無法「同時」包含經理的直屬員工及其員工。
* 此規則無法與任何其他成員資格規則結合。

### <a name="create-an-all-users-rule"></a>建立「所有使用者」規則

您建立的群組可以包含使用成員資格規則的租用戶內的所有使用者。 未來在租用戶中新增或移除使用者時，系統會自動調整群組的成員資格。

使用採用 -ne 運算子與 null 值的單一運算式來建構「所有使用者」規則。 此規則會將 B2B 來賓使用者，以及成員使用者新增到群組。

```
user.objectid -ne null
```

### <a name="create-an-all-devices-rule"></a>建立「所有裝置」規則

您建立的群組可以包含使用成員資格規則的租用戶內的所有裝置。 未來在租用戶中新增或移除裝置時，系統會自動調整群組的成員資格。

使用採用 -ne 運算子與 null 值的單一運算式來建構「所有裝置」規則：

```
device.objectid -ne null
```

### <a name="extension-properties-and-custom-extension-properties"></a>擴充屬性和自訂擴充屬性

支援以擴充屬性和自訂屬性作為動態成員資格規則中的字串屬性。 擴充屬性會從內部部署 Windows Server AD 進行同步處理，並採用 "ExtensionAttributeX" 格式，其中 X 等於 1-15。 以下是使用擴充屬性 (attribute) 作為屬性 (property) 的規則範例：

```
(user.extensionAttribute15 -eq "Marketing")
```

自訂擴充屬性會從內部部署 Windows Server AD 或從連線的 SaaS 應用程式進行同步處理，而且格式為 `user.extension_[GUID]__[Attribute]`，其中：

* 對於在 Azure AD 中建立屬性的應用程式而言，[GUID] 是其在 Azure AD 中的唯一識別碼
* [Attribute] 是屬性建立時的名稱

以下是使用自訂擴充屬性的規則範例：

```
user.extension_c272a57b722d4eb29bfe327874ae79cb__OfficeNumber -eq "123"
```

使用 [Graph 總管] 查詢使用者的屬性並搜尋屬性名稱，即可在目錄中找到自訂屬性名稱。

## <a name="rules-for-devices"></a>裝置的規則

您也可以建立規則以在群組中選取成員資格的裝置物件。 您不能同時讓使用者和裝置成為群組成員。 可以使用下列裝置屬性。

 裝置屬性  | 值 | 範例
 ----- | ----- | ----------------
 accountEnabled | true false | (device.accountEnabled -eq true)
 displayName | 任何字串值 |(device.displayName -eq "Rob Iphone”)
 deviceOSType | 任何字串值 | (device.deviceOSType -eq "iPad") -or (device.deviceOSType -eq "iPhone")
 deviceOSVersion | 任何字串值 | (device.OSVersion -eq "9.1")
 deviceCategory | 有效的裝置類別名稱 | (device.deviceCategory -eq "BYOD")
 deviceManufacturer | 任何字串值 | (device.deviceManufacturer -eq "Samsung")
 deviceModel | 任何字串值 | (device.deviceModel -eq "iPad Air")
 deviceOwnership | 個人、公司、未知 | (device.deviceOwnership -eq "Company")
 domainName | 任何字串值 | (device.domainName -eq "contoso.com")
 enrollmentProfileName | Apple 裝置註冊設定檔或 Windows Autopilot 設定檔名稱 | (device.enrollmentProfileName -eq "DEP iPhones")
 isRooted | true false | (device.isRooted -eq true)
 managementType | MDM (適用於行動裝置)<br>PC (適用於 Intune PC 代理程式管理的電腦) | (device.managementType -eq "MDM")
 organizationalUnit | 符合內部部署 Active Directory 所設定的組織單位名稱的任何字串值 | (device.organizationalUnit -eq "US PCs")
 deviceId | 有效的 Azure AD 裝置識別碼 | (device.deviceId -eq "d4fe7726-5966-431c-b3b8-cddc8fdb717d")
 objectId | 有效的 Azure AD 物件識別碼 |  (device.objectId -eq 76ad43c9-32c5-45e8-a272-7b58b58f596d")

## <a name="next-steps"></a>後續步驟

這些文章提供有關 Azure Active Directory 中群組的其他資訊。

* [查看現有的群組](../fundamentals/active-directory-groups-view-azure-portal.md)
* [建立新群組並新增成員](../fundamentals/active-directory-groups-create-azure-portal.md)
* [管理群組的設定](../fundamentals/active-directory-groups-settings-azure-portal.md)
* [管理群組的成員資格](../fundamentals/active-directory-groups-membership-azure-portal.md)
* [管理群組中使用者的動態規則](groups-dynamic-membership.md)
