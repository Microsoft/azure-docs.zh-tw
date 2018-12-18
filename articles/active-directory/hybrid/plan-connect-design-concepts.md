---
title: Azure AD Connect：設計概念 |Microsoft Docs
description: 本主題詳細說明特定的實作設計領域
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 4114a6c0-f96a-493c-be74-1153666ce6c9
ms.service: active-directory
ms.custom: azure-ad-connect
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: Identity
ms.date: 08/10/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: f85726f7058c783b6e369126532e40ceecc3c846
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46308177"
---
# <a name="azure-ad-connect-design-concepts"></a>Azure AD Connect：設計概念
本文件旨在說明 Azure AD Connect 實作設計期間必須考量的領域。 本文件是特定領域的深入探討，而在其他文件中也會簡短描述這些概念。

## <a name="sourceanchor"></a>sourceAnchor
sourceAnchor 屬性的定義是 *在物件存留期間不會變更的屬性*。 它可將物件唯一識別為內部部署和 Azure AD 中的相同物件。 此屬性也稱為 **immutableId** ，兩個名稱可交換使用。

在本文件中，「不可變 (亦即無法變更)」這個詞非常重要。 由於此屬性的值在設定之後就無法變更，所以請務必挑選支援您案例的設計。

此屬性用於下列案例︰

* 在建置新的同步處理引擎伺服器，或在災害復原案例後進行重建時，這個屬性會連結 Azure AD 中的現有物件與內部部署的物件。
* 如果您從僅限雲端的身分識別移至已同步處理的身分識別模型，則這個屬性可讓物件「完全比對」Azure AD 中的現有物件與內部部署的物件。
* 如果您使用同盟，則這個屬性會與 **userPrincipalName** 一起用於宣告來唯一識別使用者。

本主題只討論與使用者相關的 sourceAnchor。 相同的規則適用於所有的物件類型，但僅限於通常有此問題的使用者。

### <a name="selecting-a-good-sourceanchor-attribute"></a>選取良好的 sourceAnchor 屬性
此屬性值必須遵循下列規則：

* 長度少於 60 個字元
  * 系統會將 a-z、A-Z 或 0-9 以外的字元編碼並計為 3 個字元
* 不包含特殊字元︰&#92; ! # $ % & * + / = ? ^ &#96; { } | ~ < > ( ) ' ; : , [ ] " \@ _
* 必須是全域唯一的
* 必須是字串、整數或二進位
* 不應以使用者的名稱為基礎，因為這些可能會有所變更
* 不應區分大小寫，並避免可能會因大小寫而改變的值
* 建立物件時應該要予以指派

如果選取的 sourceAnchor 不是字串類型，則 Azure AD Connect 會將此屬性值進行 Base64Encode 處理，以確保不會出現特殊字元。 如果您使用 ADFS 以外的其他同盟伺服器，請確定您的伺服器也能夠將此屬性進行 Base64Encode 處理。

sourceAnchor 屬性會區分大小寫。 "JohnDoe" 與 "johndoe" 是不同的值。 但是您不應該擁有兩個只有大小寫有差異的不同物件。

如果您有單一內部部署樹系，應該使用的屬性為 **objectGUID**。 這也是您在 Azure AD Connect 中使用快速設定時所用的屬性，而且也是 DirSync 所用的屬性。

如果您有多個樹系，而且不會在樹系和網域之間移動使用者，則 **objectGUID** 是適合使用的屬性 (即使在此情況下)。

如果您在樹系和網域之間移動使用者，則必須尋找不會變更的屬性，或在移動時可隨使用者移動的屬性。 建議的方法是引入綜合的屬性。 可保存 GUID 之類項目的屬性也可能適用。 在物件建立期間，會建立新的 GUID 建立並於使用者加上戳記。 可以在同步處理引擎伺服器中建立自訂同步規則，以根據 **objectGUID** 建立這個值，然後在 ADDS 中更新選取的屬性。 當您移動物件時，請務必同時複製此值的內容。

另一個解決方案是挑選您知道不會變更的現有屬性。 常用的屬性包括 **employeeID**。 如果您考慮使用含有字母的屬性，請確定屬性值的大小寫 (大寫與小寫) 沒機會變更。 不該使用的不合適屬性，包括含使用者姓名的屬性。 結婚或離婚時，此名稱可能會變更，所以不適用於此屬性。 這也就是不可能在 Azure AD Connect 安裝精靈中選取 **userPrincipalName**、**mail** 和 **targetAddress** 之類屬性的原因之一。 這些屬性也會包含 "\@" 字元，而 sourceAnchor 中不允許此字元。

### <a name="changing-the-sourceanchor-attribute"></a>變更 sourceAnchor 屬性
在 Azure AD 中建立物件並同步處理身分識別之後，無法變更 sourceAnchor 屬性值。

基於這個理由，下列限制適用於 Azure AD Connect：

* 只能在初始安裝期間設定 sourceAnchor 屬性。 如果您重新執行安裝精靈，這個選項會是唯讀選項。 如果您需要變更此設定，就必須解除安裝後再重新安裝。
* 如果您安裝其他 Azure AD Connect 伺服器，您必須選取如先前所用的相同 sourceAnchor 屬性。 如果您稍早已使用 DirSync 並移至 Azure AD Connect，則必須使用 **objectGUID** ，因為這是 DirSync 所用的屬性。
* 如果 sourceAnchor 值在物件匯出至 Azure AD 之後變更，則 Azure AD Connect Sync 會擲回錯誤，並在修正問題且於來源目錄中將 sourceAnchor 變回之前，不允許對此物件進行任何其他變更。

## <a name="using-ms-ds-consistencyguid-as-sourceanchor"></a>使用 ms-DS-ConsistencyGuid 作為 sourceAnchor
根據預設，Azure AD Connect (1.1.486.0 版或較舊版本) 會使用 objectGUID 來作為 sourceAnchor 屬性。 ObjectGUID 是由系統所產生。 您並無法在建立內部部署 AD 物件時指定它的值。 如 [sourceAnchor](#sourceanchor) 一節所述，在某些情況下，您需要指定 sourceAnchor 值。 如果您遇到這類情況，則必須使用可設定的 AD 屬性 (例如，ms-DS-ConsistencyGuid) 來作為 sourceAnchor 屬性。

Azure AD Connect (1.1.524.0 版和更新版本) 現在可協助您使用 ms-DS-ConsistencyGuid 來作為 sourceAnchor 屬性。 在使用此功能時，Azure AD Connect 會自動將同步處理規則設定為︰

1. 使用 ms-DS-ConsistencyGuid 來作為使用者物件的 sourceAnchor 屬性。 若為其他物件類型，則要使用 ObjectGUID。

2. 對於未填入 ms-DS-ConsistencyGuid 屬性的給定內部部署 AD 使用者物件，Azure AD Connect 會將其 objectGUID 值反向寫入內部部署 Active Directory 中的 ms-DS-ConsistencyGuid 屬性。 該物件在填入 ms-DS-ConsistencyGuid 屬性之後，Azure AD Connect 便會將其匯出至 Azure AD。

>[!NOTE]
> 在內部部署 AD 物件匯入到 Azure AD Connect (也就是，匯入到 AD 連接器空間並投影到 Metaverse) 之後，您就無法再變更其 sourceAnchor 值。 若要對給定的內部部署 AD 物件指定 sourceAnchor 值，請先設定其 ms-DS-ConsistencyGuid 屬性，再將其匯入到 Azure AD Connect。

### <a name="permission-required"></a>所需權限
若要讓這項功能生效，用來與內部部署 Active Directory 同步的 AD DS 帳戶，必須獲得內部部署 Active Directory 中 ms-DS-ConsistencyGuid 屬性的寫入權限。

### <a name="how-to-enable-the-consistencyguid-feature---new-installation"></a>如何啟用 ConsistencyGuid 功能 - 新安裝
您可以在新安裝期間啟用 ConsistencyGuid 作為 sourceAnchor。 本節詳述快速和自訂安裝。

  >[!NOTE]
  > 在新安裝期間，只有較新版本的 Azure AD Connect (1.1.524.0 和更新版本) 才支援使用 ConsistencyGuid 作為 sourceAnchor。

### <a name="how-to-enable-the-consistencyguid-feature"></a>如何啟用 ConsistencyGuid 功能
目前來說，只有在安裝新的 Azure AD Connect 時，才能啟用此功能。

#### <a name="express-installation"></a>快速安裝
使用快速模式來安裝 Azure AD Connect 時，Azure AD Connect 精靈會使用下列邏輯，自動決定最適合作為 sourceAnchor 屬性的 AD 屬性︰

* 首先，Azure AD Connect 精靈會向 Azure AD 租用戶進行查詢，以擷取之前的 Azure AD Connect 安裝 (如果有的話) 中作為 sourceAnchor 屬性的 AD 屬性。 如果可以取得這項資訊，Azure AD Connect 就會使用相同的 AD 屬性。

  >[!NOTE]
  > 在安裝期間，只有較新版本的 Azure AD Connect (1.1.524.0 和更新版本) 才會在 Azure AD 租用戶中儲存有關所使用 sourceAnchor 屬性的資訊。 較舊版本的 Azure AD Connect 則不會這麼做。

* 若未提供所使用之 sourceAnchor 屬性的相關資訊，則精靈會檢查內部部署 Active Directory 中的 ms-DS-ConsistencyGuid 屬性狀態。 如果目錄中沒有任何物件設定了此屬性，精靈就會使用 ms-DS-ConsistencyGuid 來作為 sourceAnchor 屬性。 如果目錄中有一或多個物件設定了此屬性，則精靈會認為其他應用程式正在使用此屬性，因此此屬性不適合作為 sourceAnchor 屬性...

* 在此情況下，精靈會轉為使用 objectGUID 來作為 sourceAnchor 屬性。

* 在決定好 sourceAnchor 屬性之後，精靈會將此資訊儲存在 Azure AD 租用戶中。 未來在安裝 Azure AD Connect 時，系統便會使用此資訊。

在快速安裝完成之後，精靈會通知您它已挑選哪個屬性來作為 sourceAnchor 屬性。

![精靈的通知指出它已挑選 AD 屬性來作為 sourceAnchor](./media/plan-connect-design-concepts/consistencyGuid-01.png)

#### <a name="custom-installation"></a>自訂安裝
使用自訂模式來安裝 Azure AD Connect 時，Azure AD Connect 精靈會提供兩個選項供您設定 sourceAnchor 屬性︰

![自訂安裝 - sourceAnchor 設定](./media/plan-connect-design-concepts/consistencyGuid-02.png)

| 設定 | 說明 |
| --- | --- |
| 讓 Azure 為我管理來源錨點 | 如果您想要 Azure AD 為您挑選屬性，請選取此選項。 如果您選取此選項，Azure AD Connect 精靈會套用[快速安裝期間所使用的相同邏輯來選取 sourceAnchor 屬性](#express-installation)。 和快速安裝一樣，在自訂安裝完成之後，精靈會通知您它已挑選哪個屬性來作為 sourceAnchor 屬性。 |
| 特定的屬性 | 如果您希望指定現有的 AD 屬性作為 sourceAnchor 屬性，請選取此選項。 |

### <a name="how-to-enable-the-consistencyguid-feature---existing-deployment"></a>如何啟用 ConsistencyGuid 功能 - 現有部署
如果您的現有 Azure AD Connect 部署使用 objectGUID 作為「來源錨點」屬性，則可以改為將它切換成使用 ConsistencyGuid。

>[!NOTE]
> 只有較新版本的 Azure AD Connect (1.1.552.0 和更新版本) 才支援從 ObjectGuid 切換成 ConsistencyGuid 以作為「來源錨點」屬性。

從 objectGUID 切換至 ConsistencyGuid 作為「來源錨點」屬性：

1. 啟動 Azure AD Connect 精靈，然後按一下 [設定] 以移至 [工作] 畫面。

2. 選取 [設定來源錨點] 工作選項，然後按一下 [下一步]。

   ![啟用現有部署的 ConsistencyGuid - 步驟 2](./media/plan-connect-design-concepts/consistencyguidexistingdeployment01.png)

3. 輸入 Azure AD 系統管理員認證，然後按一下 [下一步]。

4. Azure AD Connect 精靈會分析內部部署 Active Directory 中的 ms-DS-ConsistencyGuid 屬性狀態。 如果未在目錄的任何物件上設定此屬性，則 Azure AD Connect 會結束，而且其他應用程式目前未使用此屬性，可安全地使用它作為「來源錨點」屬性。 選取 [下一步] 以繼續操作。

   ![啟用現有部署的 ConsistencyGuid - 步驟 4](./media/plan-connect-design-concepts/consistencyguidexistingdeployment02.png)

5. 在 [準備設定] 畫面上，按一下 [設定] 進行設定變更。

   ![啟用現有部署的 ConsistencyGuid - 步驟 5](./media/plan-connect-design-concepts/consistencyguidexistingdeployment03.png)

6. 設定完成之後，精靈會指出 ms-DS-ConsistencyGuid 現在用作「來源錨點」屬性。

   ![啟用現有部署的 ConsistencyGuid - 步驟 6](./media/plan-connect-design-concepts/consistencyguidexistingdeployment04.png)

在分析期間 (步驟 4)，如果目錄中有一或多個物件設定此屬性，則精靈會認為其他應用程式正在使用此屬性，並傳回如下圖所示的錯誤。 如果您先前已在主要 Azure AD Connect 伺服器上啟用 ConsistencyGuid 功能，而且正在嘗試對暫存伺服器執行相同動作，則也會發生此錯誤。

![啟用現有部署的 ConsistencyGuid - 錯誤](./media/plan-connect-design-concepts/consistencyguidexistingdeploymenterror.png)

 如果您確定屬性不是供其他現有應用程式使用，您可以指定 **/SkipLdapSearchcontact**，重新啟動 Azure AD Connect 精靈來隱藏錯誤。 若要這樣做，在命令提示字元中執行下列命令：

```
"c:\Program Files\Microsoft Azure Active Directory Connect\AzureADConnect.exe" /SkipLdapSearch
```

### <a name="impact-on-ad-fs-or-third-party-federation-configuration"></a>對 AD FS 或第三方同盟設定的影響
如果您要使用 Azure AD Connect 來管理內部部署 AD FS 的部署，Azure AD Connect 會自動將宣告規則更新為使用同樣的 AD 屬性來作為 sourceAnchor。 這種作法可確保 ADFS 所產生的 ImmutableID 宣告會與匯出至 Azure AD 的 sourceAnchor 值保持一致。

如果您要在 Azure AD Connect 之外管理 AD FS，或是要使用第三方同盟伺服器來進行驗證，則必須手動更新 ImmutableID 宣告的宣告規則，使其與匯出到 Azure AD 的 sourceAnchor 值保持一致，詳情如[修改 AD FS 宣告規則](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-federation-management#modclaims)一節所述。 安裝完成後，精靈會傳回下列警告︰

![第三方同盟設定](./media/plan-connect-design-concepts/consistencyGuid-03.png)

### <a name="adding-new-directories-to-existing-deployment"></a>在現有部署中新增目錄
假設您已部署 Azure AD Connect 並啟用 ConsistencyGuid 功能，而現在您想要在部署中新增另一個目錄。 當您嘗試新增目錄時，Azure AD Connect 精靈會檢查目錄中 ms-DS-ConsistencyGuid 屬性的狀態。 如果目錄中有一或多個物件設定了此屬性，則精靈會認為其他應用程式正在使用此屬性，並傳回如下圖所示的錯誤。 如果您確定現有應用程式並沒有使用此屬性，則請連絡支援人員，以了解該如何隱藏此錯誤。

![在現有部署中新增目錄](./media/plan-connect-design-concepts/consistencyGuid-04.png)

## <a name="azure-ad-sign-in"></a>Azure AD 登入
整合您的內部部署目錄與 Azure AD 時，請務必了解同步處理設定對使用者驗證的方式有何影響。 Azure AD 使用 userPrincipalName (UPN) 來驗證使用者。 不過，當您同步處理使用者時，必須小心選擇要用於 userPrincipalName 值的屬性。

### <a name="choosing-the-attribute-for-userprincipalname"></a>選擇 userPrincipalName 的屬性
當您選取屬性以便提供要用於 Azure 的 UPN 值時，應確保

* 屬性值符合 UPN 語法 (RFC 822)，其格式應該是 username\@domain
* 這些值的尾碼符合 Azure AD 中其中一個已驗證的自訂網域

在快速設定中，屬性的假定選擇會是 userPrincipalName。 如果 userprincipalname 屬性不包含您希望使用者用於登入 Azure 的值，則必須選擇 [自訂安裝] 。

### <a name="custom-domain-state-and-upn"></a>自訂網域狀態和 UPN
請務必確保 UPN 尾碼有已驗證的網域。

John 是 contoso.com 中的使用者。 在將使用者同步至 Azure AD 目錄 contoso.onmicrosoft.com 之後，您希望 John 使用內部部署 UPN john\@contoso.com 來登入 Azure。 若要這樣做，您必須將 contoso.com 新增為 Azure AD 中的自訂網域並加以驗證，才能開始同步處理使用者。 舉例來說，如果 John 的 UPN 尾碼是 contoso.com，不符合 Azure AD 中已驗證的網域，則 Azure AD 會以 contoso.onmicrosoft.com 取代 UPN 尾碼。

### <a name="non-routable-on-premises-domains-and-upn-for-azure-ad"></a>無法路由傳送的內部部署網域與 Azure AD 的 UPN
有些組織有無法路由傳送的網域，例如 contoso.local 或簡單單一標籤網域，例如 contoso。 您無法確認在 Azure AD 中無法路由傳送的網域。 Azure AD Connect 可以僅同步至 Azure AD 中已驗證的網域。 當您建立 Azure AD 目錄時，它會建立可路由傳送的網域，而該網域會成為 Azure AD 的預設網域，例如 contoso.onmicrosoft.com。 因此，如果您不想要同步至預設的 .onmicrosoft.com 網域，則必須在此類案例中驗證所有其他可路由傳送的網域。

如需有關如何新增和驗證網域的詳細資訊，請參閱 [將您的自訂網域名稱新增至 Azure Active Directory](../active-directory-domains-add-azure-portal.md) 。

Azure AD Connect 會偵測您是否在無法路由傳送的網域環境中執行，並且會適當地警告您不要繼續進行快速設定。 如果您是在不可路由傳送的網域中操作，則使用者的 UPN 可能也有無法路由傳送的尾碼。 例如，如果您是在 contoso.local 之下執行，Azure AD Connect 會建議您使用自訂設定，而不是使用快速設定。 使用自訂設定，在使用者同步至 Azure AD 之後，您能夠指定應做為 UPN 以供登入 Azure 的屬性。

## <a name="next-steps"></a>後續步驟
深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
