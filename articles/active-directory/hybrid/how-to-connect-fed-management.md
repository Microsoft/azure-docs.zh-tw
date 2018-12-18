---
title: Azure AD Connect - AD FS 管理和自訂 | Microsoft Docs
description: 使用 Azure AD Connect 進行 AD FS 管理，以及使用 Azure AD Connect 和 PowerShell 的使用者 AD FS 登入經驗的自訂。
keywords: AD FS, ADFS, AD FS 管理, AAD Connect, 連線, 登入, AD FS 自訂, 修復信任, O365, 同盟, 信賴憑證者
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 2593b6c6-dc3f-46ef-8e02-a8e2dc4e9fb9
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/18/2017
ms.component: hybrid
ms.author: billmath
ms.custom: seohack1
ms.openlocfilehash: 5a36c239e296d6f393c49932f1a369d8aae80b83
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46304677"
---
# <a name="manage-and-customize-active-directory-federation-services-by-using-azure-ad-connect"></a>使用 Azure AD Connect 管理和自訂 Active Directory Federation Services
本文說明如何使用 Azure Active Directory (Azure AD) Connect 管理及自訂 Active Directory Federation Services (AD FS)。 它也包含您可能需要進行以完整設定 AD FS 伺服器陣列的其他常見 AD FS 工作。

| 話題 | 涵蓋內容 |
|:--- |:--- |
| **管理 AD FS** | |
| [修復信任](#repairthetrust) |如何修復與 Office 365 的同盟信任。 |
| [使用替代登入識別碼與 Azure AD 建立同盟關係](#alternateid) | 使用替代登入識別碼設定同盟  |
| [新增 AD FS 伺服器](#addadfsserver) |如何使用額外的 AD FS 伺服器擴充 AD FS 伺服器陣列。 |
| [新增 AD FS Web 應用程式 Proxy 伺服器](#addwapserver) |如何使用其他 Web 應用程式 Proxy (WAP) 伺服器展開 AD FS 陣列。 |
| [新增同盟網域](#addfeddomain) |如何新增同盟網域。 |
| [更新 SSL 憑證](how-to-connect-fed-ssl-update.md)| 如何更新 AD FS 伺服器陣列的 SSL 憑證。 |
| **自訂 AD FS** | |
| [新增自訂公司標誌或圖例](#customlogo) |如何使用公司標誌與圖例自訂 AD FS 登入頁面。 |
| [新增登入說明](#addsignindescription) |如何新增登入頁面說明。 |
| [修改 AD FS 宣告規則](#modclaims) |如何修改各種同盟案例的 AD FS 宣告。 |

## <a name="manage-ad-fs"></a>管理 AD FS
您可以使用 Azure AD Connect 精靈，以最少使用者介入的形式執行各種 AD FS 相關工作。 執行精靈來完成安裝 Azure AD Connect 之後，您可以再次執行精靈，以執行其他工作。

## <a name="repairthetrust"></a>修復信任 
您可以使用 Azure AD Connect 檢查 AD FS 和 Azure AD trust 目前的健全狀況，並採取適當的動作來修復信任。 請遵循下列步驟來修復您的 Azure AD 和 AD FS 信任。

1. 從其他工作的清單中選取 [修復 AAD 和 ADFS 信任]  。
   ![](./media/how-to-connect-fed-management/RepairADTrust1.PNG)

2. 在 [連線到 Azure AD] 頁面上，提供 Azure AD 的全域系統管理員認證，然後按 [下一步]。
   ![連接至 Azure AD](./media/how-to-connect-fed-management/RepairADTrust2.PNG)

3. 在 [遠端存取認證]  頁面上，輸入網域系統管理員的認證。

   ![遠端存取認證](./media/how-to-connect-fed-management/RepairADTrust3.PNG)

    在按 [下一步] 之後，Azure AD Connect 會檢查憑證健康情況並顯示任何問題。

    ![憑證的狀態](./media/how-to-connect-fed-management/RepairADTrust4.PNG)

    [準備設定] 頁面會顯示為了修復信任，將執行的動作清單。

    ![準備設定](./media/how-to-connect-fed-management/RepairADTrust5.PNG)

4. 按一下 [安裝]  以修復信任。

> [!NOTE]
> Azure AD Connect 只可以對自我簽署的憑證進行修復或採取動作。 Azure AD Connect 無法修復第三方憑證。

## <a name="alternateid"></a>使用 AlternateID 來與 Azure AD 建立同盟關係 
建議您讓內部部署使用者主體名稱 (UPN) 和雲端使用者主體名稱保持相同。 如果內部部署 UPN 使用無法路由傳送的網域 (例如︰ Contoso.local)，或是由於本機應用程式相依性而無法變更，我們會建議您設定替代登入識別碼。 替代登入識別碼可讓您設定登入體驗，讓使用者可以透過其 UPN 以外的屬性 (例如 mail) 來進行登入。 Azure AD Connect 預設會選擇 Active Directory 中的 userPrincipalName 屬性來作為使用者主體名稱。 如果您選擇任何其他屬性來作為使用者主體名稱，而且您使用 AD FS 來建立同盟，則 Azure AD Connect 會就替代登入識別碼對 AD FS 進行設定。 選擇不同屬性來作為使用者主體名稱的範例如下所示︰

![替代識別碼屬性的選擇](./media/how-to-connect-fed-management/attributeselection.png)

AD FS 替代登入識別碼的設定作業包含兩個主要步驟︰
1. **設定正確的發行宣告集**︰Azure AD 信賴憑證者信任中的發行宣告規則，會修改為使用選取的 UserPrincipalName 屬性來作為使用者的替代識別碼。
2. **在 AD FS 設定中啟用替代登入識別碼**︰AD FS 設定會進行更新，讓 AD FS 可以使用替代識別碼在適當樹系中查詢使用者。 此設定支援 Windows Server 2012 R2 (含 KB2919355) 或更新版本上的 AD FS。 如果 AD FS 伺服器是 2012 R2，Azure AD Connect 會檢查是否有必要的 KB 存在。 如果未偵測到必要 KB，則系統會在設定完成之後顯示警告，如下所示︰

    ![2012 R2 上缺少 KB 的警告](./media/how-to-connect-fed-management/kbwarning.png)

    若要在缺少 KB 時修正設定，請安裝必要的 [KB2919355](http://go.microsoft.com/fwlink/?LinkID=396590)，然後使用[修復 AAD 與 AD FS 信任](#repairthetrust)來修復信任。

> [!NOTE]
> 如需替代識別碼以及手動設定步驟的詳細資訊，請閱讀[設定替代登入識別碼](https://technet.microsoft.com/windows-server-docs/identity/ad-fs/operations/configuring-alternate-login-id)

## <a name="addadfsserver"></a>新增 AD FS 伺服器 

> [!NOTE]
> 若要新增 AD FS 伺服器，Azure AD Connect 需要 PFX 憑證檔案。 因此，只有當您使用 Azure AD Connect 來設定 AD FS 伺服器陣列時，才可以執行這項作業。

1. 選取 [部署其他同盟伺服器]，然後按 [下一步]。

   ![其他同盟伺服器](./media/how-to-connect-fed-management/AddNewADFSServer1.PNG)

2. 在 [連線到 Azure AD] 頁面上，輸入 Azure AD 的全域系統管理員認證，然後按 [下一步]。

   ![連接至 Azure AD](./media/how-to-connect-fed-management/AddNewADFSServer2.PNG)

3. 提供網域系統管理員認證。

   ![網域系統管理員認證](./media/how-to-connect-fed-management/AddNewADFSServer3.PNG)

4. Azure AD Connect 會要求您提供您在使用 Azure AD Connect 設定您的新 AD FS 伺服器陣列時所提供的 PFX 檔案的密碼。 按一下 [輸入密碼]  以提供 PFX 檔案的密碼。

   ![憑證密碼](./media/how-to-connect-fed-management/AddNewADFSServer4.PNG)

    ![指定 SSL 憑證](./media/how-to-connect-fed-management/AddNewADFSServer5.PNG)

5. 在 [AD FS 伺服器]  頁面上，輸入要新增到 AD FS 伺服器陣列的伺服器名稱或 IP 位址。

   ![AD FS 伺服器](./media/how-to-connect-fed-management/AddNewADFSServer6.PNG)

6. 按 [下一步]，並逐步進行到最終的 [設定] 頁面。 Azure AD Connect 完成將伺服器新增至 AD FS 伺服器陣列之後，將提供您選項來驗證連線。

   ![準備設定](./media/how-to-connect-fed-management/AddNewADFSServer7.PNG)

    ![安裝完成](./media/how-to-connect-fed-management/AddNewADFSServer8.PNG)

## <a name="addwapserver"></a>新增 AD FS WAP 伺服器 

> [!NOTE]
> 若要新增 WAP 伺服器，Azure AD Connect 需要 PFX 憑證檔案。 因此，只有當您使用 Azure AD Connect 來設定 AD FS 伺服器陣列時，才可以執行這項作業。

1. 從可用的工作清單中選取 [部署 Web 應用程式 Proxy]  。

   ![部署 Web 應用程式 Proxy](./media/how-to-connect-fed-management/WapServer1.PNG)

2. 提供 Azure 全域系統管理員認證。

   ![連接至 Azure AD](./media/how-to-connect-fed-management/wapserver2.PNG)

3. 在 [指定 SSL 憑證] 頁面上，提供您在使用 Azure AD Connect 設定 AD FS 伺服器陣列時所提供之 PFX 檔案的密碼。
   ![憑證密碼](./media/how-to-connect-fed-management/WapServer3.PNG)

    ![指定 SSL 憑證](./media/how-to-connect-fed-management/WapServer4.PNG)

4. 新增要做為 WAP 伺服器的伺服器。 因為 WAP 伺服器可能不會加入網域，精靈會要求要新增之伺服器的系統管理認證。

   ![管理伺服器認證](./media/how-to-connect-fed-management/WapServer5.PNG)

5. 在 [Proxy 信任憑證]  頁面上，提供系統管理認證，以設定 Proxy 信任和存取 AD FS 伺服器陣列中的主要伺服器。

   ![Proxy 信任憑證](./media/how-to-connect-fed-management/WapServer6.PNG)

6. 在 [準備設定]  頁面上，精靈會顯示將執行的動作清單。

   ![準備設定](./media/how-to-connect-fed-management/WapServer7.PNG)

7. 按一下 [安裝]  以完成組態。 組態完成之後，精靈會提供您選項，來驗證與伺服器的連線。 按一下 [驗證]  來檢查連線能力。

   ![安裝完成](./media/how-to-connect-fed-management/WapServer8.PNG)

## <a name="addfeddomain"></a>新增同盟網域 

您可以使用 Azure AD Connect 輕鬆地新增要與 Azure AD 同盟的網域。 Azure AD Connect 會新增同盟的網域並修改宣告規則，以在您有與 Azure AD 同盟的多個網域時正確反映發行者。

1. 若要新增同盟網域，請選取工作 [新增其他 Azure AD 網域] 。

   ![其他 Azure AD 網域](./media/how-to-connect-fed-management/AdditionalDomain1.PNG)

2. 在精靈的下一個頁面上，提供 Azure AD 全域系統管理員認證。

   ![連接至 Azure AD](./media/how-to-connect-fed-management/AdditionalDomain2.PNG)

3. 在 [遠端存取認證]  頁面上，提供網域系統管理員認證。

   ![遠端存取認證](./media/how-to-connect-fed-management/additionaldomain3.PNG)

4. 在下一個頁面上，精靈會提供 Azure AD 網域的清單，以供您用來同盟您的內部部署目錄。 從清單選擇網域。

   ![Azure AD 網域](./media/how-to-connect-fed-management/AdditionalDomain4.PNG)

    選擇網域之後，精靈會提供您關於精靈將採取的進一步動作和組態影響的適當資訊。 在某些情況下，如果您選取尚未在 Azure AD 中驗證的網域，精靈將提供資訊協助您驗證網域。 如需詳細資訊，請參閱 [將您的自訂網域名稱新增至 Azure Active Directory](../active-directory-domains-add-azure-portal.md) 。

5. 按 [下一步] 。 按 [下一步]，然後 [準備設定] 頁面就會顯示 Azure AD Connect 將會執行的動作清單。 按一下 [安裝]  以完成組態。

   ![準備設定](./media/how-to-connect-fed-management/AdditionalDomain5.PNG)

> [!NOTE]
> 所新增同盟網域的使用者必須保持同步，才能夠登入 Azure AD。

## <a name="ad-fs-customization"></a>AD FS 自訂
下列各節提供在自訂 AD FS 登入頁面時，可能必須執行之某些常見工作的詳細資料。

## <a name="customlogo"></a>新增自訂公司標誌或圖例 
若要變更 [登入] 頁面上顯示的公司標誌，請使用下列 Windows PowerShell Cmdlet 和語法。

> [!NOTE]
> 建議的標誌尺寸為 260 x 35 \@ 96 dpi，檔案大小不超過 10 KB。

    Set-AdfsWebTheme -TargetName default -Logo @{path="c:\Contoso\logo.PNG"}

> [!NOTE]
> *TargetName* 是必要參數。 隨著 AD FS 釋出的預設佈景主題為指定的預設值。

## <a name="addsignindescription"></a>新增登入說明 
若要在 [登入] 頁面中新增登入頁面描述，請使用下列 Windows PowerShell Cmdlet 和語法。

    Set-AdfsGlobalWebContent -SignInPageDescriptionText "<p>Sign-in to Contoso requires device registration. Click <A href='http://fs1.contoso.com/deviceregistration/'>here</A> for more information.</p>"

## <a name="modclaims"></a>修改 AD FS 宣告規則 
AD FS 支援豐富的宣告語言，您可以用它來建立自訂宣告規則。 如需詳細資訊，請參閱 [宣告規則語言的角色](https://technet.microsoft.com/library/dd807118.aspx)。

下列各節說明如何為關於 Azure AD 和 AD FS 同盟的一些案例撰寫自訂規則。

### <a name="immutable-id-conditional-on-a-value-being-present-in-the-attribute"></a>屬性中的值出現固定 ID 條件
Azure AD Connect 可在將物件同步處理至 Azure AD 時，讓您指定要做為來源錨點的屬性。 如果自訂屬性中的值未空白，您可能會想要發出固定 ID 宣告。

例如，您可能會選取 **ms-ds-consistencyguid** 做為來源錨點的屬性，並且在該屬性具有值時發出 **ImmutableID** 做為 **ms-ds-consistencyguid**。 如果沒有該屬性的值，則發出 **objectGuid** 做為固定 ID。 您可以如下一節所述建構自訂宣告規則的集合。

**規則 1：查詢屬性**

    c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname"]
    => add(store = "Active Directory", types = ("http://contoso.com/ws/2016/02/identity/claims/objectguid", "http://contoso.com/ws/2016/02/identity/claims/msdsconsistencyguid"), query = "; objectGuid,ms-ds-consistencyguid;{0}", param = c.Value);

在此規則中，您會從 Active Directory 查詢使用者的 **ms-ds-consistencyguid** 和 **objectGuid** 值。 在 AD FS 部署中，將存放區名稱變更為適當的存放區名稱。 另外，也將依照針對 **objectGuid** 和 **ms-ds-consistencyguid** 所定義的，將宣告類型變更為您的同盟適用的宣告類型。

此外，使用**新增**而非**發出**，您可以避免為實體新增傳出發出，而可以使用這些值做為中間值。 在您建立要做為固定 ID 的值之後，您將在後續的規則中發出宣告。

**規則 2：檢查使用者是否存在 ms-ds-consistencyguid**

    NOT EXISTS([Type == "http://contoso.com/ws/2016/02/identity/claims/msdsconsistencyguid"])
    => add(Type = "urn:anandmsft:tmp/idflag", Value = "useguid");

此規則會定義稱為 **idflag** 的暫時旗標，如果沒有為使用者填入 **ms-ds-concistencyguid**，則此旗標會設為 **useguid**。 背後邏輯是實際上 AD FS 不允許空的宣告。 因此，當您在「規則 1」中新增宣告 http://contoso.com/ws/2016/02/identity/claims/objectguid 和 http://contoso.com/ws/2016/02/identity/claims/msdsconsistencyguid 時，只有已為使用者填入值時，您最終才會具有 **msdsconsistencyguid** 宣告。 如果未填入，AD FS 會看到它將具有空值，並因此立即捨棄。 所有物件都會有 **objectGuid**，因此執行規則 1 之後，宣告一律會在該處。

**規則 3：發出 ms-ds-consistencyguid 為固定 ID (如果有的話)**

    c:[Type == "http://contoso.com/ws/2016/02/identity/claims/msdsconsistencyguid"]
    => issue(Type = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier", Value = c.Value);

這是隱含的 **存在** 檢查。 如果宣告的值存在，則發出做為固定 ID。 上述範例使用 **nameidentifier** 宣告。 您必須為您的環境中的固定 ID 將此宣告變更為適當的宣告類型。

**規則 4：發出 objectGuid 做為固定 ID，如果未出現 ms-ds-consistencyGuid**

    c1:[Type == "urn:anandmsft:tmp/idflag", Value =~ "useguid"]
    && c2:[Type == "http://contoso.com/ws/2016/02/identity/claims/objectguid"]
    => issue(Type = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier", Value = c2.Value);

在這項規則中，您只會檢查暫存旗標 **idflag**。 您必須根據宣告值來決定是否要發出宣告。

> [!NOTE]
> 這些規則的順序很重要。

### <a name="sso-with-a-subdomain-upn"></a>使用子網域 UPN 的 SSO

您可以使用 Azure AD Connect 新增多個要同盟的網域，如 [新增新的同盟網域](how-to-connect-fed-management.md#addfeddomain)所述。 Azure AD Connect 1.1.553.0 版和最新版本會為 issuerID 自動建立正確的宣告規則。 如果您無法使用 Azure AD Connect 1.1.553.0 版或最新版本，建議您使用 [Azure AD RPT 宣告規則](https://aka.ms/aadrptclaimrules)工具，針對 Azure AD 信賴憑證者的信任產生和設定正確的宣告規則。

## <a name="next-steps"></a>後續步驟
深入了解 [使用者登入選項](plan-connect-user-signin.md)。
