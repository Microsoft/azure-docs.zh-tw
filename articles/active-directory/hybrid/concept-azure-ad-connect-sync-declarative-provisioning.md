---
title: Azure AD Connect：了解宣告式佈建 | Microsoft Docs
description: 說明 Azure AD Connect 中的宣告式佈建組態模型。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: cfbb870d-be7d-47b3-ba01-9e78121f0067
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/13/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 9242ffc0c87ee9f314745463b8287ad7531a982d
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46310287"
---
# <a name="azure-ad-connect-sync-understanding-declarative-provisioning"></a>Azure AD Connect 同步處理：了解宣告式佈建
本主題說明 Azure AD Connect 中的組態模型。 此模型稱為宣告式佈建，它可讓您輕鬆地進行組態變更。 本主題中所述的許多項目都是進階的，而且在大部分客戶案例中並非必要。

## <a name="overview"></a>概觀
宣告式佈建正在處理來自來源連接的目錄中的物件，並決定應如何將物件和屬性從來源轉換到目標。 物件會在同步處理管線中進行處理，而輸入和輸出規則的管線都相同。 輸入規則是從連接器空間至 Metaverse，而輸出規則是從 Metaverse 至連接器空間。

![同步處理管線](./media/concept-azure-ad-connect-sync-declarative-provisioning/sync1.png)  

管線有數個不同的模組。 每個模組負責物件同步處理中的一個概念。

![同步處理管線](./media/concept-azure-ad-connect-sync-declarative-provisioning/pipeline.png)  

* 資料：來源物件
* [範圍](#scope)：尋找所有適用的同步處理規則
* [聯結](#join)：判斷連接器空間和 Metaverse 之間的關聯性
* [轉換](#transform)：計算屬性應如何轉換和流動
* [優先順序](#precedence)：解決衝突的屬性貢獻
* 目標：目標物件

## <a name="scope"></a>影響範圍
範圍模組會評估物件，並判斷在範圍中且應納入處理的規則。 視物件上的屬性值而定，不同的同步處理規則會評估為在範圍中。 例如，沒有 Exchange 信箱的已停用使用者會有不同於具有信箱的已啟用使用者的規則。  
![影響範圍](./media/concept-azure-ad-connect-sync-declarative-provisioning/scope1.png)  

範圍可定義為群組和子句。 子句位於群組中。 邏輯 AND 使用於群組中的所有子句之間。 例如，(department =IT AND country = Denmark)。 邏輯 OR 使用於群組之間。

![範圍](./media/concept-azure-ad-connect-sync-declarative-provisioning/scope2.png)  
此圖中的範圍應該讀為 (department = IT AND country = Denmark) OR (country=Sweden)。 如果群組 1 或 2 群組評估為 true，規則便在範圍中。

範圍模組支援下列作業。

| 作業 | 說明 |
| --- | --- |
| EQUAL、NOTEQUAL |評估此值是否等於屬性值的字串比較。 若為多重值屬性，請參閱 ISIN 和 ISNOTIN。 |
| LESSTHAN、LESSTHAN_OR_EQUAL |評估此值是否小於屬性值的字串比較。 |
| CONTAINS、NOTCONTAINS |評估是否可在屬性值中找到此值的字串比較。 |
| STARTSWITH、NOTSTARTSWITH |評估此值是否在屬性值開頭的字串比較。 |
| ENDSWITH、NOTENDSWITH |評估此值是否在屬性值結尾的字串比較。 |
| GREATERTHAN、GREATERTHAN_OR_EQUAL |評估此值是否大於屬性值的字串比較。 |
| ISNULL、ISNOTNULL |評估物件是否不存在此屬性。 如果屬性不存在，因而為 null，規則便在範圍中。 |
| ISIN、ISNOTIN |判斷此值是否存在於定義的屬性中。 此作業是 EQUAL 和 NOTEQUAL 的多重值變化。 屬性應該是多重值的屬性，而如果可以在任何屬性值中找到此值，規則便在範圍中。 |
| ISBITSET、ISNOTBITSET |評估是否已設定特定的位元。 例如，可用來評估 userAccountControl 中的位元，以查看使用者已啟用或停用。 |
| ISMEMBEROF、ISNOTMEMBEROF |此值應該包含連接器空間中群組的 DN。 如果物件是指定群組的成員，規則便在範圍中。 |

## <a name="join"></a>聯結
同步處理管線中的聯結模組負責尋找來源中物件和目標中物件之間的關聯性。 在輸入規則上，此關聯性是連接器空間中的物件找到對 Metaverse 中物件的關聯性。  
![聯結 cs 與 mv](./media/concept-azure-ad-connect-sync-declarative-provisioning/join1.png)  
目標在於查看應該相關聯的 Metaverse 中是否已經有一個物件 (由另一個連接器建立)。 比方說，在帳戶-資源樹系中，帳戶樹系中的使用者應該與資源樹系中的使用者聯結。

聯結大多數使用於輸入規則，以將連接器空間物件與相同的 Metaverse 物件聯結在一起。

聯結會定義為一或多個群組。 在群組中，您有一些子句。 邏輯 AND 使用於群組中的所有子句之間。 邏輯 OR 使用於群組之間。 群組的處理順序為從上而下。 當一個群組在目標中恰巧找到一個相符的物件，則不會評估任何其他聯結規則。 如果找到零個或多個物件，則處理會繼續下一個規則群組。 基於這個理由，應該最先建立最明確的規則，而最後建立比較模糊的規則。  
![聯結定義](./media/concept-azure-ad-connect-sync-declarative-provisioning/join2.png)  
此圖中的聯結會從上而下處理。 首先，同步處理管線會查看是否有相符的 employeeID。 如果沒有，第二個規則會查看帳戶名稱是否可用來將物件聯結在一起。 如果也不相符，則第三個 (最後一個) 規則會使用使用者名稱尋找更模糊的相符項目。

如果所有聯結規則經評估後沒有完全相符的項目，則會使用 [說明] 頁面上的 [連結類型]。 如果此選項設定為 [佈建] ，則目標中會建立新的物件。  
![佈建或聯結](./media/concept-azure-ad-connect-sync-declarative-provisioning/join3.png)  

一個物件應該只有一個同步處理規則具有在範圍中的聯結規則。 如果有多個同步處理規則定義了聯結，則會發生錯誤。 優先順序不會用於解決聯結衝突。 物件必須具有在範圍中的聯結規則，屬性才能以相同的輸入/輸出方向流動。 如果您需要讓屬性以輸入和輸出方式流向相同的物件，則聯結必須具有輸入和輸出同步處理規則。

輸出聯結嘗試將物件佈建到目標連接器空間時，會有特殊的行為。 DN 屬性用於先嘗試反向聯結。 如果目標連接器空間中已經有相同 DN 的物件，則物件會聯結起來。

當新的同步處理規則進入範圍時，只會評估聯結模組一次。 物件若已聯結，即使不再符合聯結準則，也不會取消聯結。 如果您想要取消物件的聯結，則聯結物件的同步處理規則必須超出範圍。

### <a name="metaverse-delete"></a>Metaverse 刪除
只要有一個在範圍中的同步處理規則，Metaverse 物件的 [連結類型] 就會維持設定為 [佈建] 或 [StickyJoin]。 StickyJoin 使用於不允許連接器將新物件佈建至 Metaverse 時 ，但若已聯結該物件，則必須先在來源中加以刪除，才能刪除 Metaverse 物件。

刪除 Metaverse 物件後，所有與標示要 [佈建]  的輸出同步處理規則相關聯的物件都會標示要刪除。

## <a name="transformations"></a>轉換
轉換用來定義屬性應如何從來源流動到目標。 流程可以有下列其中一種 **流程類型**︰直接、常數或運算式。 直接流程，讓屬性值依現狀流動，而不進行其他轉換。 常數值可設定指定的值。 運算式會使用宣告式佈建運算式語言來表示應該如何轉換。 如需運算式語言的詳細資料，請參閱 [了解宣告式佈建運算式語言](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md) 主題。

![佈建或聯結](./media/concept-azure-ad-connect-sync-declarative-provisioning/transformations1.png)  

[套用一次]  核取方塊會定義只能在最初建立物件時設定的屬性。 例如，此組態可用來設定新使用者物件的初始密碼。

### <a name="merging-attribute-values"></a>合併屬性值
在屬性流程中，有一個設定可決定是否應該從數個不同的連接器合併多重值屬性。 預設值為 [Update] ，表示應採用具有最高優先順序的同步處理規則。

![合併類型](./media/concept-azure-ad-connect-sync-declarative-provisioning/mergetype.png)  

另外還有 **Merge** 和 **MergeCaseInsensitive**。 這些選項可讓您合併來自不同來源的值。 例如，它可用於合併來自數個不同樹系的成員或 proxyAddresses 屬性。 當您使用此選項時，物件範圍中的所有同步處理規則必須使用相同的合併類型。 您不能定義從一個連接器 **Update** 和從另一個連接器 **Merge**。 如果您嘗試，您會收到錯誤。

**Merge** 和 **MergeCaseInsensitive** 之間的差異在於處理重複屬性值的方式。 同步處理引擎可確保不會將重複的值插入目標屬性中。 使用 [MergeCaseInsensitive] ，就不會出現只有大小寫差異的重複值。 例如，您不應該在目標屬性中同時看到 SMTP:bob@contoso.com 和 smtp:bob@contoso.com。  只會查看只可能出現大小寫差異的確切值和多個值。

**Replace** 選項與 **Update** 相同，但未使用。

### <a name="control-the-attribute-flow-process"></a>控制屬性流程程序
將多個輸入同步處理規則設定為貢獻於相同的 Metaverse 屬性時，則會使用優先順序來決定獲得採用的規則。 具有最高優先順序 (最小數值) 的同步處理規則將會貢獻值。 輸出規則的情況一樣。 具有最高優先順序的同步處理規則會獲得採用，並貢獻值給已連接的目錄。

在某些情況下，同步處理規則應該判斷其他規則的行為方式，而不是貢獻值。 在此情況下會使用一些特殊的常值。

對於輸入同步處理規則，常值 **NULL** 可用來指出流程沒有要貢獻的值。 具有較低優先順序的其他規則可以貢獻一個值。 如果沒有規則提供值，則會移除 Metaverse 屬性。 對於輸出規則，如果 **NULL** 是處理完所有同步處理規則後的最終值，則會在已連接的目錄中移除此值。

常值 **AuthoritativeNull** 類似 **NULL**，但差別在於沒有較低優先順序的規則可以貢獻值。

屬性流程也可以使用 **IgnoreThisFlow**。 意思上類似於 NULL，表示沒有要貢獻的項目。 差別在於它不會移除目標中已經存在的值。 好像屬性流程未曾出現一樣。

下列是一個範例：

在 *Out to AD - User Exchange hybrid* 可以找到下列流程：  
`IIF([cloudSOAExchMailbox] = True,[cloudMSExchSafeSendersHash],IgnoreThisFlow)`  
此運算式的意思是︰如果使用者信箱位於 Azure AD 中，則將屬性從 Azure AD 傳送至 AD。 如果並非如此，請勿將任何項目送回 Active Directory。 在此情況下，它會在 AD 中保留現有的值。

### <a name="importedvalue"></a>ImportedValue
函式 ImportedValue 與其他所有函式都不同，其屬性名稱必須以引號 (而非方括號) 括住：  
`ImportedValue("proxyAddresses")`。

通常在同步處理期間，屬性會使用預期的值，即使它尚未匯出或在匯出期間 (「協定塔的頂端」) 收到錯誤。 輸入同步處理會假設尚未到達已連接目錄的屬性最後還是會到達。 在某些情況下，請務必只同步處理已連接目錄所確認的值 (「全像圖和差異匯入協定塔」)。

在現成可用的同步處理規則 In from AD – User Common from Exchange 中可以找到此函式的範例。 在混合式 Exchange 中，由 Exchange Online 新增的值只有在確認已成功匯出該值時，才能同步處理：  
`proxyAddresses` <- `RemoveDuplicates(Trim(ImportedValue("proxyAddresses")))`

## <a name="precedence"></a>優先順序
當數個同步處理規則嘗試將相同的屬性貢獻給目標時，則會使用優先順序值來決定獲得採用的規則。 具有最高優先順序 (最小數值) 的規則將會在衝突中貢獻此屬性。

![合併類型](./media/concept-azure-ad-connect-sync-declarative-provisioning/precedence1.png)  

此排序方式可針對一小部分的物件，用來定義更精確的屬性流程。 例如，現成可用的規則會確定已啟用帳戶 (**User AccountEnabled**) 的屬性優先於其他帳戶的屬性。

可以定義連接器之間的優先順序。 這可讓具有較佳資料的連接器先貢獻值。

### <a name="multiple-objects-from-the-same-connector-space"></a>相同連接器空間中的多個物件
如果在聯結至相同 Metaverse 物件的相同連接器空間中有數個物件，則必須調整優先順序。 如果數個物件都是在相同同步處理規則的範圍中，則同步處理引擎便無法判斷優先順序。 應該貢獻值給 Metaverse 的來源物件模稜兩可。 即使來源中的屬性具有相同的值，此組態仍會回報為模稜兩可。  
![多個物件聯結至相同的 mv 物件](./media/concept-azure-ad-connect-sync-declarative-provisioning/multiple1.png)  

在此案例中，您需要變更同步處理規則的範圍，讓來源物件在範圍中有不同的同步處理規則。 這可讓您定義不同的優先順序。  
![多個物件聯結至相同的 mv 物件](./media/concept-azure-ad-connect-sync-declarative-provisioning/multiple2.png)  

## <a name="next-steps"></a>後續步驟
* 如需運算式語言的詳細資訊，請參閱 [了解宣告式佈建運算式](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md)。
* 如需了解如何立即使用宣告式佈建，請參閱 [了解預設組態](concept-azure-ad-connect-sync-default-configuration.md)。
* 如需了解如何使用宣告式佈建進行實際變更，請參閱 [如何變更預設組態](how-to-connect-sync-change-the-configuration.md)。
* 如需了解使用者與連絡人的共同運作方式，請繼續閱讀 [了解使用者和連絡人](concept-azure-ad-connect-sync-user-and-contacts.md)。

**概觀主題**

* [Azure AD Connect 同步處理：了解及自訂同步處理](how-to-connect-sync-whatis.md)
* [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)

**參考主題**

* [Azure AD Connect 同步處理：函式參考](reference-connect-sync-functions-reference.md)
