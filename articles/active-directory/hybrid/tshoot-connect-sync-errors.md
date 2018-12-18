---
title: Azure AD Connect：針對同步處理期間的錯誤進行疑難排解 | Microsoft Docs
description: 說明如何針對 Azure AD Connect 同步處理期間發生的錯誤進行疑難排解。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
ms.assetid: 2209d5ce-0a64-447b-be3a-6f06d47995f8
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/31/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 5180aa935ed5ab1a263c8dd002c2009ec89d5d86
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46304837"
---
# <a name="troubleshooting-errors-during-synchronization"></a>針對同步處理期間的錯誤進行疑難排解
將身分識別資料從 Windows Server Active Directory (AD DS) 同步處理至 Azure Active Directory (Azure AD) 時，可能會發生錯誤。 本文提供不同類型的同步處理錯誤概觀、某些可能導致這些錯誤的案例，以及修正錯誤的可能方式。 本文包含常見的錯誤類型，不一定涵蓋所有可能的錯誤。

 本文假設讀者已熟悉 [Azure AD 與 Azure AD Connect 的基礎設計概念](plan-connect-design-concepts.md)。

使用最新版的 Azure AD Connect \(2016年 8 月或更新版本\)，在 [Azure 入口網站](https://aka.ms/aadconnecthealth)中可取得「同步處理錯誤」報告做為 Azure AD Connect Health for Sync 的一部分。

從 2016 年 9 月 1 日開始，預設會針對所有「新的」Azure Active Directory 租用戶啟用 [Azure Active Directory 重複屬性恢復](how-to-connect-syncservice-duplicate-attribute-resiliency.md)功能。 這項功能會針對未來幾個月內的現有租用戶自動啟用。

Azure AD Connect 會從其保持同步的目錄執行三種類型的作業︰匯入、同步處理及匯出。 錯誤可能會發生於所有作業中。 本文主要著重於在匯出至 Azure AD 期間的錯誤。

## <a name="errors-during-export-to-azure-ad"></a>匯出至 Azure AD 期間的錯誤
下一節說明使用 Azure AD 連接器匯出至 Azure AD 的作業期間內可能發生的各類型同步處理錯誤。 此連接器可經由 "contoso.onmicrosoft.com" 格式的名稱加以識別。
匯出至 Azure AD 期間發生的錯誤，表示 Azure AD connect \(同步處理引擎\) 在 Azure Active Directory 上嘗試的\(新增、更新、刪除等\)作業失敗。

![匯出錯誤概觀](./media/tshoot-connect-sync-errors/Export_Errors_Overview_01.png)

## <a name="data-mismatch-errors"></a>資料不符錯誤
### <a name="invalidsoftmatch"></a>InvalidSoftMatch
#### <a name="description"></a>說明
* 當 Azure AD Connect \(同步處理引擎\)指示 Azure Active Directory 新增或更新物件時，Azure AD 會比對使用 **sourceAnchor** 屬性的連入物件與 Azure AD 中物件的 **immutableId** 屬性。 此種比對稱為**完全比對**。
* 當 Azure AD **找不到**有任何物件符合 **immutableId** 屬性與連入物件的 **sourceAnchor** 屬性時，在佈建新物件之前，它會改回使用 ProxyAddresses 和 UserPrincipalName 屬性來尋找相符項目。 此種比對稱為**大致比對**。 「大致比對」的設計用來比對 Azure AD 中已存在的物件 (也就是源自 Azure AD) 與在同步處理期間進行新增/更新的新物件 (代表內部部署上的相同實體 (使用者、群組))。
* **InvalidSoftMatch** 發生於完全比對找不到任何相符物件，**而且**大致比對找到相符物件，但該物件具有的 immutableId 值不同於連入物件的 SourceAnchor 值時，就表示相符的物件已經與內部部署 Active Directory 上的另一個物件同步處理。

換句話說，若要讓大致比對得以運作，要進行大致比對的物件不得有任何的 immutableId 值。 如果任何已設定 immutableId 值的物件完全比對失敗，但滿足大致比對準則，則作業會導致 InvalidSoftMatch 同步處理錯誤。

Azure Active Directory 結構描述不允許兩個或多個物件的下列屬性具有相同的值。 \(這不是詳盡的清單。\)

* ProxyAddresses
* UserPrincipalName
* onPremisesSecurityIdentifier
* ObjectId

> [!NOTE]
> [Azure AD 屬性重複屬性恢復](how-to-connect-syncservice-duplicate-attribute-resiliency.md)功能也正當作 Azure Active Directory 的預設行為推出。  讓 Azure AD 能夠更彈性地處理內部部署 AD 環境中重複的 ProxyAddresses 和 UserPrincipalName 屬性，將會減少 Azure AD Connect (以及其他同步處理用戶端) 所看到的同步處理錯誤數目。 這項功能無法修復重複錯誤。 因此資料仍然需要修復。 但可讓您佈建由於 Azure AD 中的重複值而無法佈建的新物件。 這也會減少傳回至同步處理用戶端的同步處理錯誤數目。
> 如果您的租用戶已啟用這項功能，則不會在新物件的佈建過程中看到 InvalidSoftMatch 同步處理錯誤。
>
>

#### <a name="example-scenarios-for-invalidsoftmatch"></a>InvalidSoftMatch 的範例案例
1. 內部部署 Active Directory 中存在兩個或多個具有相同 ProxyAddresses 屬性值的物件。 只有一個會在 Azure AD 中佈建。
2. 內部部署 Active Directory 中存在兩個或多個具有相同 userPrincipalName 屬性值的物件。 只有一個會在 Azure AD 中佈建。
3. 內部部署 Active Directory 中新增的物件具有與 Azure Active Directory 中現有物件相同的 ProxyAddresses 屬性值。 在內部部署新增的物件並未佈建於 Azure Active Directory 中。
4. 內部部署 Active Directory 中新增的物件具有與 Azure Active Directory 中的帳戶相同的 userPrincipalName 屬性值。 此物件並未佈建於 Azure Active Directory 中。
5. 同步處理的帳戶已從樹系 A 移到樹系 B。Azure AD Connect (同步處理引擎) 使用 ObjectGUID 屬性來計算 SourceAnchor。 在樹系移動之後，SourceAnchor 的值會不同。 新物件 (來自樹系 B) 與 Azure AD 中的現有物件同步處理失敗。
6. 同步處理的物件會意外從內部部署 Active Directory 中刪除，而在 Active Directory 中針對相同的實體 (例如使用者) 建立新物件，而不需刪除 Azure Active Directory 中的帳戶。 新的帳戶與現有的 Azure AD 物件同步處理失敗。
7. 已將 Azure AD Connect 解除安裝後再重新安裝。 在重新安裝期間，選擇了不同的屬性作為 SourceAnchor。 所有先前同步處理的物件都停止與 InvalidSoftMatch 錯誤同步處理。

#### <a name="example-case"></a>範例案例︰
1. **Bob Smith** 是 Azure Active Directory 中已從 *contoso.com* 的內部部署 Active Directory 同步處理的使用者
2. Bob Smith 的 **UserPrincipalName** 已設定為 **bobs@contoso.com**。
3. **"abcdefghijklmnopqrstuv=="** 是 Azure AD Connect 使用 Bob Smith 的 **objectGUID** 從內部部署 Active Directory 計算的 **SourceAnchor**，這是 Azure Active Directory 中 Bob Smith 的 **immutableId**。
4. Bob 也具有 **proxyAddresses** 屬性的下列值︰
   * smtp: bobs@contoso.com
   * smtp: bob.smith@contoso.com
   * **smtp: bob@contoso.com**
5. 新使用者 **Bob Taylor** 已新增至內部部署 Active Directory。
6. Bob Taylor 的 **UserPrincipalName** 已設為 **bobt@contoso.com**。
7. **"abcdefghijkl0123456789==""** 是 Azure AD Connect 使用 Bob Taylor 的 **objectGUID** 從內部部署 Active Directory 計算的 **sourceAnchor**。 Bob Taylor 的物件尚未同步處理至 Azure Active Directory。
8. Bob Taylor 具有 proxyAddresses 屬性的下列值︰
   * smtp: bobt@contoso.com
   * smtp: bob.taylor@contoso.com
   * **smtp: bob@contoso.com**
9. 在同步處理期間，Azure AD Connect 會辨識內部部署 Active Directory 中的 Bob Taylor 新增，並要求 Azure AD 進行相同的變更。
10. Azure AD 會先執行完全比對。 也就是說，它會搜尋是否有任何物件的 immutableId 等於 "abcdefghijkl0123456789=="。 完全比對會失敗，因為 Azure AD 中沒有其他物件具有該 immutableId。
11. Azure AD 會接著嘗試大致比對 Bob Taylor。 也就是說，它將會搜尋是否有任何物件的 proxyAddresses 等於上述三個值，包括 smtp: bob@contoso.com
12. Azure AD 會找到 Bob Smith 的物件以比對大致相符準則。 但此物件 immutableId 的值等於 "abcdefghijklmnopqrstuv=="。 這表示此物件已從內部部署 Active Directory 中的另一個物件同步處理。 因此，Azure AD 無法大致比對這些物件，並會導致 **InvalidSoftMatch** 同步處理錯誤。

#### <a name="how-to-fix-invalidsoftmatch-error"></a>如何修正 InvalidSoftMatch 錯誤
InvalidSoftMatch 錯誤的最常見原因是兩個具有不同 SourceAnchor \(immutableId\) 的物件具有相同的 ProxyAddresses 和/或 UserPrincipalName 屬性值，而這兩個物件使用於 Azure AD 上的大致比對程序。 若要修正無效的大致比對：

1. 找出造成錯誤的重複 proxyAddresses、userPrincipalName 或其他屬性值。 同時找出哪兩個 \( 或多個\) 物件與衝突有關。 [Azure AD Connect Health for Sync](https://aka.ms/aadchsyncerrors) 所產生的報告可協助您找出這兩個物件。
2. 找出哪個物件應該繼續擁有重複的值，以及哪個物件不該如此。
3. 移除物件中不該具有該值的重複值。 您應該在物件的來源目錄中進行變更。 在某些情況下，您可能需要刪除其中一個衝突的物件。
4. 如果您在內部部署 AD 中進行變更，請讓 Azure AD Connect 同步處理變更。

Azure AD Connect Health 進行同步處理的同步處理錯誤報告每隔 30 分鐘就會更新一次，並包含最近同步處理嘗試的錯誤。

> [!NOTE]
> 根據定義，不得在物件的存留期間變更 ImmutableId 。 如果設定 Azure AD Connect 時未牢記上述清單中的一些案例，您最後可能會遇到以下情況：Azure AD Connect 針對 AD 物件計算不同的 SourceAnchor 值，其代表具有您要繼續使用之現有 Azure AD 物件的相同實體 (相同的使用者/群組/連絡人等)。
>
>

#### <a name="related-articles"></a>相關文章
* [重複或無效的屬性會防止 Office 365 進行目錄同步作業](https://support.microsoft.com/kb/2647098)

### <a name="objecttypemismatch"></a>ObjectTypeMismatch
#### <a name="description"></a>說明
當 Azure AD 嘗試大致比對兩個物件時，兩個不同「物件類型」(例如使用者、群組、連絡人等) 的物件可能具有相同的屬性值來執行大致比對。 Azure AD 中不允許這些屬性重複，因為作業可能會導致 "ObjectTypeMismatch" 同步處理錯誤。

#### <a name="example-scenarios-for-objecttypemismatch-error"></a>ObjectTypeMismatch 錯誤的示範案例
* 在 Office 365 中建立擁有郵件功能的安全性群組。 系統管理員會使用與 Office 365 群組相同的 ProxyAddresses 屬性值，在內部部署 AD (尚未同步處理至 Azure AD) 中新增使用者或連絡人。

#### <a name="example-case"></a>範例案例
1. 系統管理員在 Office 365 中針對稅務部門建立擁有郵件功能的新安全性群組，並為其提供電子郵件地址 tax@contoso.com。 將 **smtp: tax@contoso.com** 的 ProxyAddresses 屬性值指派給這個群組
2. 新使用者會加入 Contoso.com，並針對 proxyAddress 為 **smtp: tax@contoso.com** 的內部部署使用者建立帳戶
3. 當 Azure AD Connect 將同步處理新的使用者帳戶時，它會收到 "ObjectTypeMismatch" 錯誤。

#### <a name="how-to-fix-objecttypemismatch-error"></a>如何修正 ObjectTypeMismatch 錯誤
ObjectTypeMismatch 錯誤的最常見原因是兩個不同類型 (使用者、群組、連絡人等) 的物件具有相同的 ProxyAddresses 屬性值。 若要修正 ObjectTypeMismatch：

1. 找出造成錯誤的重複 proxyAddresses (或其他屬性) 值。 同時找出哪兩個 \( 或多個\) 物件與衝突有關。 [Azure AD Connect Health for Sync](https://aka.ms/aadchsyncerrors) 所產生的報告可協助您找出這兩個物件。
2. 找出哪個物件應該繼續擁有重複的值，以及哪個物件不該如此。
3. 移除物件中不該具有該值的重複值。 請注意，您應該在物件源自的目錄中進行變更。 在某些情況下，您可能需要刪除其中一個衝突的物件。
4. 如果您在內部部署 AD 中進行變更，請讓 Azure AD Connect 同步處理變更。 Azure AD Connect Health for Sync 內的同步處理錯誤報告會每隔 30 分鐘更新一次，包含最近同步處理嘗試的錯誤。

## <a name="duplicate-attributes"></a>重複的屬性
### <a name="attributevaluemustbeunique"></a>AttributeValueMustBeUnique
#### <a name="description"></a>說明
Azure Active Directory 結構描述不允許兩個或多個物件的下列屬性具有相同的值。 也就是強制 Azure AD 中的每個物件在指定的執行個體具有這些屬性的唯一值。

* ProxyAddresses
* UserPrincipalName

如果 Azure AD Connect 嘗試新增物件，或以指派給 Azure Active Directory 中另一個物件的上述屬性值更新現有的物件，作業會導致 "AttributeValueMustBeUnique" 同步處理錯誤。

#### <a name="possible-scenarios"></a>可能的案例︰
1. 重複的值指派給已同步處理的物件，但該物件與另一個已同步處理的物件發生衝突。

#### <a name="example-case"></a>範例案例︰
1. **Bob Smith** 是 Azure Active Directory 中已從 contoso.com 的內部部署 Active Directory 同步處理的使用者
2. Bob Smith 的內部部署 **UserPrincipalName** 已設定為 **bobs@contoso.com**。
3. Bob 也具有 **proxyAddresses** 屬性的下列值︰
   * smtp: bobs@contoso.com
   * smtp: bob.smith@contoso.com
   * **smtp: bob@contoso.com**
4. 新使用者 **Bob Taylor** 已新增至內部部署 Active Directory。
5. Bob Taylor 的 **UserPrincipalName** 已設為 **bobt@contoso.com**。
6. **Bob Taylor** 具有 **ProxyAddresses** 屬性的下列值。i. smtp: bobt@contoso.com ii. smtp: bob.taylor@contoso.com
7. Bob Taylor 物件已成功與 Azure AD 同步處理。
8. 系統管理員決定使用下列的值更新 Bob Taylor 的 **ProxyAddresses** 屬性︰i. **smtp: bob@contoso.com**
9. Azure AD 將嘗試使用上述的值更新 Bob Taylor 在 Azure AD 中的物件，但該作業將會失敗，因為該 ProxyAddresses 值已指派給 Bob Smith，因而導致 "AttributeValueMustBeUnique" 錯誤。

#### <a name="how-to-fix-attributevaluemustbeunique-error"></a>如何修正 AttributeValueMustBeUnique 錯誤
AttributeValueMustBeUnique 錯誤的最常見原因是兩個具有不同 SourceAnchor \(immutableId\) 的物件具有相同的 ProxyAddresses 和/或 UserPrincipalName 屬性值。 若要修正 AttributeValueMustBeUnique 錯誤：

1. 找出造成錯誤的重複 proxyAddresses、userPrincipalName 或其他屬性值。 同時找出哪兩個 \( 或多個\) 物件與衝突有關。 [Azure AD Connect Health for Sync](https://aka.ms/aadchsyncerrors) 所產生的報告可協助您找出這兩個物件。
2. 找出哪個物件應該繼續擁有重複的值，以及哪個物件不該如此。
3. 移除物件中不該具有該值的重複值。 請注意，您應該在物件源自的目錄中進行變更。 在某些情況下，您可能需要刪除其中一個衝突的物件。
4. 如果您在內部部署 AD 中進行變更，請讓 Azure AD Connect 同步處理變更，以便修正錯誤。

#### <a name="related-articles"></a>相關文章
-[重複或無效的屬性會防止 Office 365 進行目錄同步作業](https://support.microsoft.com/kb/2647098)

## <a name="data-validation-failures"></a>資料驗證失敗
### <a name="identitydatavalidationfailed"></a>IdentityDataValidationFailed
#### <a name="description"></a>說明
Azure Active Directory 會先對資料本身強制執行各種限制，然後才允許該資料寫入目錄中。 這些限制是為了確保使用者在使用相依於這項資料的應用程式時，能獲得最棒的體驗。

#### <a name="scenarios"></a>案例
a. UserPrincipalName 屬性值具有無效/不支援的字元。
b. UserPrincipalName 屬性不符合所需的格式。

#### <a name="how-to-fix-identitydatavalidationfailed-error"></a>如何修正 IdentityDataValidationFailed 錯誤
a. 確定 userPrincipalName 屬性具有支援的字元和所需的格式。

#### <a name="related-articles"></a>相關文章
* [準備透過 Office 365 目錄同步作業佈建使用者](https://support.office.com/en-us/article/Prepare-to-provision-users-through-directory-synchronization-to-Office-365-01920974-9e6f-4331-a370-13aea4e82b3e)

### <a name="federateddomainchangeerror"></a>FederatedDomainChangeError
#### <a name="description"></a>說明
這種情況會在使用者的 UserPrincipalName 尾碼從一個同盟網域變更為另一個同盟網域時，導致發生 **"FederatedDomainChangeError"** 同步處理錯誤。

#### <a name="scenarios"></a>案例
對於已同步處理的使用者，UserPrincipalName 尾碼從一個同盟網域變更為另一個內部部署的同盟網域。 例如，UserPrincipalName = bob@contoso.com 已變更為 UserPrincipalName = bob@fabrikam.com。

#### <a name="example"></a>範例
1. Bob Smith (contoso.com 的帳戶) 新增為 Active Directory 中具有 UserPrincipalName bob@contoso.com 的新使用者
2. Bob 移至 Contoso.com 的不同部門 (名為 Fabrikam.com)，而他的 UserPrincipalName 變更為 bob@fabrikam.com
3. Contoso.com 和 fabrikam.com 網域都是使用 Azure Active Directory 的同盟網域。
4. Bob 的 userPrincipalName 並不會更新，而導致發生 "FederatedDomainChangeError" 同步處理錯誤。

#### <a name="how-to-fix"></a>修正方式
如果使用者的 UserPrincipalName 尾碼從 bob@**contoso.com** 更新為 bob@**fabrikam.com**，其中 **contoso.com** 與 **fabrikam.com** 都是**同盟網域**，則遵循這些步驟來修正同步處理錯誤

1. 將 Azure AD 中使用者的 UserPrincipalName 從 bob@contoso.com 更新為 bob@contoso.onmicrosoft.com。 您可以使用以下 PowerShell 命令搭配 Azure AD PowerShell 模組：`Set-MsolUserPrincipalName -UserPrincipalName bob@contoso.com -NewUserPrincipalName bob@contoso.onmicrosoft.com`
2. 允許下一個同步處理週期嘗試進行同步處理。 這次同步處理將會成功，而且 Bob 的 UserPrincipalName 會如預期般更新為 bob@fabrikam.com。

#### <a name="related-articles"></a>相關文章
* [在您將使用者帳戶的 UPN 變更為使用不同的同盟網域後，Azure Active Directory 同步作業工具未同步處理變更](https://support.microsoft.com/help/2669550/changes-aren-t-synced-by-the-azure-active-directory-sync-tool-after-you-change-the-upn-of-a-user-account-to-use-a-different-federated-domain)

## <a name="largeobject"></a>LargeObject
### <a name="description"></a>說明
當屬性超過 Azure Active Directory 結構描述所設定的允許大小限制、長度限制或計數限制時，同步處理作業會導致 **LargeObject** 或 **ExceededAllowedLength** 同步處理錯誤。 此錯誤通常會發生於下列屬性

* userCertificate
* userSMIMECertificate
* thumbnailPhoto
* proxyAddresses

### <a name="possible-scenarios"></a>可能的案例
1. Bob 的 userCertificate 屬性儲存太多指派給 Bob 的憑證。 這些可能包含已過期的舊版憑證。 固定限制為 15 個憑證。 如需有關如何處理 userCertificate 屬性引起之 LargeObject 錯誤的詳細資訊，請參閱[處理 userCertificate 屬性所造成的 LargeObject 錯誤](tshoot-connect-largeobjecterror-usercertificate.md)一文。
2. Bob 的 userSMIMECertificate 屬性儲存太多指派給 Bob 的憑證。 這些可能包含已過期的舊版憑證。 固定限制為 15 個憑證。
3. Bob 在 Active Directory 中設定的 thumbnailPhoto 太大，以致於無法在 Azure AD 中進行同步處理。
4. 在 Active Directory 中自動填入 ProxyAddresses 屬性期間，一個物件被指派太多 ProxyAddresses。

### <a name="how-to-fix"></a>修正方式
1. 確定造成錯誤的屬性在允許的限制內。

## <a name="related-links"></a>相關連結
* [在 Active Directory 管理中心找出 Active Directory 物件](https://technet.microsoft.com/library/dd560661.aspx)
* [如何使用 Azure Active Directory PowerShell 查詢 Azure Active Directory 中的物件](https://msdn.microsoft.com/library/azure/jj151815.aspx)
