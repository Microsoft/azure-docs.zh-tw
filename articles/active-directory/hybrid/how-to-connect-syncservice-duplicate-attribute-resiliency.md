---
title: 身分識別同步處理和重複屬性恢復功能 | Microsoft Docs
description: 在目錄同步處理期間如何使用 Azure AD Connect 來處理具有 UPN 或 ProxyAddress 衝突之物件的新行為。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 537a92b7-7a84-4c89-88b0-9bce0eacd931
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/15/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: f8fe86712b59a896d1f431ef677f592ced382db8
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306057"
---
# <a name="identity-synchronization-and-duplicate-attribute-resiliency"></a>身分識別同步處理和重複屬性恢復功能
「重複屬性恢復功能」是 Azure Active Directory 中的一項功能，可在執行 Microsoft 的其中一個同步處理工具時，用來消除 **UserPrincipalName** 和 **ProxyAddress** 衝突所造成的不便。

在指定之 Azure Active Directory 租用戶的所有「使用者」、「群組」或「連絡人」物件中，這兩個屬性通常必須是唯一的。

> [!NOTE]
> 只有使用者可以擁有 UPN。
> 
> 

這項功能賦予了同步管線雲端部分的新行為，因此對於任何 Microsoft 同步處理產品 (包括 Azure AD Connect、DirSync 和 MIM + 連接器) 而言，此功能不限於特定用戶端並與用戶端相關。 在本文件中，通用詞彙「同步用戶端」將用來代表上述任何一項產品。

## <a name="current-behavior"></a>目前的行為
如果嘗試佈建的新物件具有違反此唯一性條件約束的 UPN 或 ProxyAddress 值，則 Azure Active Directory 不會建立該物件。 同樣地，如果以非唯一的 UPN 或 ProxyAddress 更新物件，則更新會失敗。 同步用戶端會在每個匯出週期重試佈建嘗試或更新，在衝突解決前會繼續失敗。 每次嘗試時都會產生錯誤報告電子郵件，並由同步用戶端記錄一個錯誤。

## <a name="behavior-with-duplicate-attribute-resiliency"></a>重複屬性恢復功能的行為
Azure Active Directory 並不是完全無法佈建或更新具有重複屬性的物件，而是會「隔離」違反唯一性條件約束的重複屬性。 如果佈建時需要此屬性 (例如 UserPrincipalName)，則服務會指派預留位置值。 這些暫存值的格式為  
“***<OriginalPrefix>+<4DigitNumber>@<InitialTenantDomain>.onmicrosoft.com***”。  
如果不需要此屬性 (例如 **ProxyAddress**)，Azure Active Directory 就會隔離衝突屬性，然後繼續建立或更新物件。

隔離屬性時，衝突相關資訊會以舊版行為中使用的相同錯誤報告電子郵件傳送。 不過，此資訊只會出現在錯誤報告中一次，發生隔離時，將不會繼續記錄在未來的電子郵件中。 此外，此物件已成功匯出，所以同步用戶端不會記錄錯誤，而且不會在後續的同步週期中重試建立 / 更新作業。

為了支援此行為，[使用者]、[群組] 和 [連絡人] 物件類別已加入新屬性︰  
**DirSyncProvisioningErrors**

這是多重值的屬性，用來儲存正常新增時會違反唯一性條件約束的衝突屬性。 Azure Active Directory 中已啟用背景計時器工作，該工作會每小時執行以尋找已解決的重複屬性衝突，然後從隔離區中自動移除上述屬性。

### <a name="enabling-duplicate-attribute-resiliency"></a>啟用重複屬性恢復功能
「重複屬性恢復功能」將會是所有 Azure Active Directory 租用戶的新預設行為。 針對所有在 2016 年 8 月 22 日或之後第一次啟用同步處理的租用戶，預設都會開啟這項功能。 在此日期之前啟用同步功能的租用戶則會分批啟用此功能。 這項功能推出將從 2016 年 9 月開始，每個租用戶的技術通知連絡人將會收到電子郵件通知，其中會指出啟用功能的特定日期。

> [!NOTE]
> 一旦開啟「重複屬性恢復功能」，即無法予以停用。

若要檢查是否已為您的租用戶啟用此功能，只要下載最新版的 Azure Active Directory PowerShell 模組並執行下列程式碼即可︰

`Get-MsolDirSyncFeatures -Feature DuplicateUPNResiliency`

`Get-MsolDirSyncFeatures -Feature DuplicateProxyAddressResiliency`

> [!NOTE]
> 在為租用戶開啟該功能之前，您無法再使用 Set-MsolDirSyncFeature Cmdlet 來主動啟用重複屬性復原功能。 若要測試該功能，您需要建立新的 Azure Active Directory 租用戶。

## <a name="identifying-objects-with-dirsyncprovisioningerrors"></a>識別具有 DirSyncProvisioningErrors 的物件
目前有兩個方法可識別因為重複屬性衝突而發生錯誤的物件：Azure Active Directory PowerShell 和 Office 365 系統管理入口網站。 未來計劃擴充其他以入口網站為基礎的報告。

### <a name="azure-active-directory-powershell"></a>Azure Active Directory PowerShell
對於本主題中的 PowerShell Cmdlet，下列項目為真︰

* 下列所有 Cmdlet 都會區分大小寫。
* 一律包含 **–ErrorCategory PropertyConflict** 。 目前沒有其他類型的 **ErrorCategory**，但可能會在未來加以擴充。

首先，執行 **Connect-MsolService** 並輸入租用戶系統管理員的認證。

接著，使用下列 Cmdlet 和運算子，以不同的方式檢視錯誤︰

1. [檢視全部](#see-all)
2. [依屬性類型](#by-property-type)
3. [依衝突的值](#by-conflicting-value)
4. [使用字串搜尋](#using-a-string-search)
5. [排序](#sorted)
6. [以有限的數量或全部](#in-a-limited-quantity-or-all)

#### <a name="see-all"></a>檢視全部
連線之後，若要查看租用戶執行中屬性佈建錯誤的一般清單︰

`Get-MsolDirSyncProvisioningError -ErrorCategory PropertyConflict`

這會產生如下所示的結果︰  
 ![Get-MsolDirSyncProvisioningError](./media/how-to-connect-syncservice-duplicate-attribute-resiliency/1.png "Get-MsolDirSyncProvisioningError")  

#### <a name="by-property-type"></a>依屬性類型
若要依屬性類型查看錯誤，請搭配 **UserPrincipalName** 或 **ProxyAddresses** 引數新增 **-PropertyName** 旗標︰

`Get-MsolDirSyncProvisioningError -ErrorCategory PropertyConflict -PropertyName UserPrincipalName`

或

`Get-MsolDirSyncProvisioningError -ErrorCategory PropertyConflict -PropertyName ProxyAddresses`

#### <a name="by-conflicting-value"></a>依衝突的值
若要查看與特定屬性相關的錯誤，請新增 **-PropertyValue** 旗標 (新增此旗標時，必須一併使用 **-PropertyName**)：

`Get-MsolDirSyncProvisioningError -ErrorCategory PropertyConflict -PropertyValue User@domain.com -PropertyName UserPrincipalName`

#### <a name="using-a-string-search"></a>使用字串搜尋
若要進行廣泛的字串搜尋，請使用 **-SearchString** 旗標。 此旗標可以與上述所有旗標獨立使用，但一律為必要項目的 **-ErrorCategory PropertyConflict**除外︰

`Get-MsolDirSyncProvisioningError -ErrorCategory PropertyConflict -SearchString User`

#### <a name="in-a-limited-quantity-or-all"></a>以有限的數量或全部
1. **MaxResults <Int>** 可用來將查詢的值數量限制為特定數目。
2. **All** 可用於確保在有大量錯誤的情況下擷取所有的結果。

`Get-MsolDirSyncProvisioningError -ErrorCategory PropertyConflict -MaxResults 5`

## <a name="office-365-admin-portal"></a>Office 365 系統管理入口網站
您可以在 Office 365 系統管理中心檢視目錄同步處理錯誤。 Office 365 入口網站中的報告只會顯示有這些錯誤的 **User** 物件。 並不會顯示「群組」和「連絡人」之間的衝突相關資訊。

![作用中使用者](./media/how-to-connect-syncservice-duplicate-attribute-resiliency/1234.png "作用中使用者")

如需有關如何在 Office 365 系統管理中心檢視目錄同步處理錯誤的指示，請參閱[在 Office 365 中識別目錄同步處理錯誤](https://support.office.com/en-us/article/Identify-directory-synchronization-errors-in-Office-365-b4fc07a5-97ea-4ca6-9692-108acab74067)。

### <a name="identity-synchronization-error-report"></a>身分識別同步處理錯誤報告
當利用這個新行為處理具有重複屬性衝突的物件時，通知會包含在標準身分識別同步處理錯誤報告電子郵件中，而該電子郵件回傳送給租用戶的技術通知連絡人。 不過，此行為有一項重大變更。 在過去，重複屬性衝突的相關資訊會包含在每個後續的錯誤報告中，直到解決衝突為止。 利用這個新行為，給定衝突的錯誤通知只會出現一次 - 在衝突的屬性遭到隔離時。

ProxyAddress 衝突的電子郵件通知範例如下所示︰  
    ![作用中使用者](./media/how-to-connect-syncservice-duplicate-attribute-resiliency/6.png "作用中使用者")  

## <a name="resolving-conflicts"></a>解決衝突
這些錯誤的疑難排解策略和解決技巧不得與過去處理重複屬性錯誤的方式不同。 唯一的差別在於計時器工作將會掃掠服務端上的租用戶，以在衝突解決後，自動將有問題的屬性新增至適當的物件。

下列文章概述各種疑難排解和解決策略︰ [重複或無效的屬性會防止在 Office 365 中進行目錄同步作業](https://support.microsoft.com/kb/2647098)。

## <a name="known-issues"></a>已知問題
沒有任何已知問題會導致資料遺失或服務降級。 其中有幾個是美觀的問題，其他問題會導致擲回標準「恢復前」重複屬性錯誤，而不是隔離衝突屬性，而別的問題會導致特定錯誤需要額外的手動修復。

**核心行為︰**

1. 具有特定屬性組態的物件會繼續收到匯出錯誤，而不是將重複屬性隔離。  
   例如︰
   
    a. 新使用者會在 AD 中以 UPN 為 **Joe@contoso.com**、ProxyAddress 為 **smtp:Joe@contoso.com** 的方式建立
   
    b. 此物件的屬性會與 ProxyAddress 為 **SMTP:Joe@contoso.com** 的現有「群組」發生衝突。
   
    c. 匯出時，會擲回「ProxyAddress 衝突」 錯誤，而不是將衝突屬性隔離。 此作業會在每個後續的同步處理週期中重試，就如同在啟用恢復功能之前一樣。
2. 如果在內部部署上建立兩個具有相同 SMTP 位址的群組，則會在第一次嘗試時佈建失敗並發生標準的重複 **ProxyAddress** 錯誤。 不過，重複值會在下一個同步處理週期時被適當隔離。

**Office 入口網站報告**：

1. UPN 衝突集中兩個物件的詳細錯誤訊息是相同的。 這表示它們都已變更 / 隔離 UPN，當事實上只有其中一個變更了資料。
2. UPN 衝突的詳細錯誤訊息會對已變更/隔離其 UPN 的使用者，顯示錯誤的 displayName。 例如︰
   
    a. **User A** 會先與 **UPN = User@contoso.com** 同步。
   
    b. 接著，會嘗試將 **User B** 與 User@contoso.comUPN =  同步。
   
    c. **User B** 的 UPN 會變更為 **User1234@contoso.onmicrosoft.com**，而 **User@contoso.com** 會新增至 **DirSyncProvisioningErrors**。
   
    d. **User B** 的錯誤訊息應指出 **User A** 已經有 **User@contoso.com** 作為 UPN，但顯示的卻是 **User B** 自己的 displayName。

**身分識別同步處理錯誤報告**：

「如何解決此問題的步驟」的連結不正確：  
    ![作用中使用者](./media/how-to-connect-syncservice-duplicate-attribute-resiliency/6.png "作用中使用者")  

它應該指向 [https://aka.ms/duplicateattributeresiliency](https://aka.ms/duplicateattributeresiliency)。

## <a name="see-also"></a>另請參閱
* [Azure AD Connect 同步處理](how-to-connect-sync-whatis.md)
* [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)
* [在 Office 365 中識別目錄同步處理錯誤](https://support.office.com/en-us/article/Identify-directory-synchronization-errors-in-Office-365-b4fc07a5-97ea-4ca6-9692-108acab74067)

