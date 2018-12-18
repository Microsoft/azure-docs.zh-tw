---
title: Azure AD Connect 多個網域
description: 本文件說明如何使用 O365 與 Azure AD 安裝及設定多個最上層網域。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: curtand
ms.assetid: 5595fb2f-2131-4304-8a31-c52559128ea4
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/31/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 8b5abe252ab9b3389680508537ea1d6f3823f910
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46305757"
---
# <a name="multiple-domain-support-for-federating-with-azure-ad"></a>與 Azure AD 同盟的多網域支援
下列文件提供與 Office 365 或 Azure AD 網域同盟時，如何使用多個頂層網域和子網域的指引。

## <a name="multiple-top-level-domain-support"></a>多個最上層網域支援
若要讓多個最上層網域與 Azure AD 同盟，您需要一些讓單一最上層網域同盟時不需要的額外組態。

當網域與 Azure AD 同盟時，系統會在 Azure 中的網域上設定幾個屬性。  其中一個重要屬性是 IssuerUri。  這個屬性是 Azure AD 用來識別權杖相關網域的 URI。  該 URI 不需要解析為任何內容，不過它必須是有效的 URI。  根據預設，Azure AD 會在內部部署 AD FS 設定中將 URI 設定為同盟服務識別碼的值。

> [!NOTE]
> 同盟服務識別碼是可唯一識別同盟服務的 URI。  同盟服務是能做為 Security Token Service 的 AD FS 執行個體。
>
>

您可以使用 PowerShell 命令 `Get-MsolDomainFederationSettings -DomainName <your domain>` 來檢視 IssuerUri。

![Get-MsolDomainFederationSettings](./media/how-to-connect-install-multiple-domains/MsolDomainFederationSettings.png)

當您加入多個頂層網域時，問題便油然而生。  例如，假設您已在 Azure AD 和內部部署環境之間設定同盟。  針對本文件，會使用 bmcontoso.com 作為網域。  現在，假設我們加入第二個頂層網域 bmfabrikam.com。

![網域](./media/how-to-connect-install-multiple-domains/domains.png)

當您嘗試將 bmfabrikam.com 網域轉換為同盟時，就會發生錯誤。  原因在於 Azure AD 具有不允許 IssuerURI 屬性針對多個網域擁有相同值的限制。  

![同盟錯誤](./media/how-to-connect-install-multiple-domains/error.png)

### <a name="supportmultipledomain-parameter"></a>SupportMultipleDomain 參數
若要解決此限制，您需要新增不同的 IssuerUri，這可以透過使用 `-SupportMultipleDomain` 參數來達成。  這個參數可搭配下列 Cmdlet 使用：

* `New-MsolFederatedDomain`
* `Convert-MsolDomaintoFederated`
* `Update-MsolFederatedDomain`

這個參數可讓 Azure AD 根據網域名稱設定 IssuerUri。  IssuerUri 在 Azure AD 的所有目錄中將是唯一的。  使用參數可讓 PowerShell 命令順利完成。

![同盟錯誤](./media/how-to-connect-install-multiple-domains/convert.png)

查看 bmfabrikam.com 網域的設定，您可以看到以下內容：

![同盟錯誤](./media/how-to-connect-install-multiple-domains/settings.png)

`-SupportMultipleDomain` 不會變更其他端點，這些端點仍設為指向 adfs.bmcontoso.com 上的同盟服務。

`-SupportMultipleDomain` 的另一個功用是確保 AD FS 系統在簽發給 Azure AD 之權杖中包含正確的簽發者值。 此值的設定方式是取得使用者 UPN 的網域部分，並將其設定為 IssuerUri 中的網域 (即 https://{upn 尾碼}/adfs/services/trust)。

因此在 Azure AD 或 Office 365 驗證期間，系統會以使用者權杖的 IssuerUri 項目來尋找 Azure AD 中的網域。  如果找不到相符項目，驗證將會失敗。

例如，如果使用者的 UPN 是 bsimon@bmcontoso.com，AD FS 所簽發之權杖中的 IssuerUri 元素將會被設定為 http://bmcontoso.com/adfs/services/trust。 此元素將會符合 Azure AD 設定，而驗證將會成功。

以下是實作此邏輯的自訂宣告規則：

    c:[Type == "http://schemas.xmlsoap.org/claims/UPN"] => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, ".+@(?<domain>.+)", "http://${domain}/adfs/services/trust/"));


> [!IMPORTANT]
> 若要在嘗試加入新網域或轉換已經存在的網域時使用 -SupportMultipleDomain 參數，您的同盟信任需要已設定為支援它們。
>
>

## <a name="how-to-update-the-trust-between-ad-fs-and-azure-ad"></a>如何更新 AD FS 與 Azure AD 之間的信任
如果您未在 AD FS 與 Azure AD 執行個體之間設定同盟信任，可能需要重新建立此信任。  原因在於，最初未使用 `-SupportMultipleDomain` 參數進行設定時，系統會將 IssuerUri 設定為預設值。  在下列螢幕擷取畫面中，您可以看到已將 IssuerUri 設定為 https://adfs.bmcontoso.com/adfs/services/trust。

如果您已成功地在 Azure AD 入口網站中加入新網域，接著嘗試使用 `Convert-MsolDomaintoFederated -DomainName <your domain>` 來轉換它，將會收到下列錯誤。

![同盟錯誤](./media/how-to-connect-install-multiple-domains/trust1.png)

如果您嘗試加入 `-SupportMultipleDomain` 參數，將會收到下列錯誤：

![同盟錯誤](./media/how-to-connect-install-multiple-domains/trust2.png)

單單嘗試針對原始網域執行 `Update-MsolFederatedDomain -DomainName <your domain> -SupportMultipleDomain` ，也會產生錯誤。

![同盟錯誤](./media/how-to-connect-install-multiple-domains/trust3.png)

使用下列步驟來加入其他最上層網域。  如果您已經加入網域且未使用 `-SupportMultipleDomain` 參數，請從移除並更新原始網域的步驟開始。  如果您尚未加入頂層網域，則可從使用 Azure AD Connect 的 PowerShell 加入網域的步驟開始。

請使用下列步驟來移除 Microsoft Online 信任，然後更新您的原始網域。

1. 在 AD FS 同盟伺服器上，開啟 [AD FS 管理] 
2. 展開左側的 [信任關係] 和 [信賴憑證者信任]
3. 刪除右側的 **Microsoft Office 365 身分識別平台** 項目。
   ![移除 Microsoft Online](./media/how-to-connect-install-multiple-domains/trust4.png)
4. 在已安裝[適用於 Windows PowerShell 的 Microsoft Azure Active Directory 模組](https://msdn.microsoft.com/library/azure/jj151815.aspx)的機器上執行下列命令：`$cred=Get-Credential`。  
5. 輸入要同盟之 Azure AD 網域的全域管理員使用者名稱和密碼。
6. 在 PowerShell 中輸入 `Connect-MsolService -Credential $cred`
7. 在 PowerShell 中輸入 `Update-MSOLFederatedDomain -DomainName <Federated Domain Name> -SupportMultipleDomain`。  此更新適用於原始網域。  所以使用上述網域後，它將會成為：`Update-MsolFederatedDomain -DomainName bmcontoso.com -SupportMultipleDomain`

使用下列步驟以透過 PowerShell 加入新的最上層網域

1. 在已安裝[適用於 Windows PowerShell 的 Microsoft Azure Active Directory 模組](https://msdn.microsoft.com/library/azure/jj151815.aspx)的機器上執行下列命令：`$cred=Get-Credential`。  
2. 輸入欲同盟之 Azure AD 網域的全域管理員使用者名稱和密碼
3. 在 PowerShell 中輸入 `Connect-MsolService -Credential $cred`
4. 在 PowerShell 中輸入 `New-MsolFederatedDomain –SupportMultipleDomain –DomainName`

使用下列步驟以透過 Azure AD Connect 加入新的最上層網域。

1. 從桌面或 [開始] 功能表啟動 Azure AD Connect
2. 選擇 [新增其他 Azure AD 網域] ![新增其他 Azure AD 網域](./media/how-to-connect-install-multiple-domains/add1.png)
3. 輸入您的 Azure AD 和 Active Directory 認證
4. 選取要設定同盟的第二個網域。
   ![新增其他 Azure AD 網域](./media/how-to-connect-install-multiple-domains/add2.png)
5. 按一下 [安裝]

### <a name="verify-the-new-top-level-domain"></a>確認新的最上層網域
藉由使用 PowerShell 命令 `Get-MsolDomainFederationSettings -DomainName <your domain>`，您可以檢視更新的 IssuerUri。  下列螢幕擷取畫面會顯示已在原始網域 http://bmcontoso.com/adfs/services/trust 上更新同盟設定

![Get-MsolDomainFederationSettings](./media/how-to-connect-install-multiple-domains/MsolDomainFederationSettings.png)

此外，已將新網域上的 IssuerUri 設定為 https://bmfabrikam.com/adfs/services/trust

![Get-MsolDomainFederationSettings](./media/how-to-connect-install-multiple-domains/settings2.png)

## <a name="support-for-subdomains"></a>對於子網域的支援
在加入子網域時，因為 Azure AD 處理網域的方式，子網域將會繼承父項的設定。  所以，IssuerUri 需要與父項相符。

因此，假設我有 bmcontoso.com，並接著加入 corp.bmcontoso.com。  來自 corp.bmcontoso.com 之使用者的 IssuerUri 必須是 **http://bmcontoso.com/adfs/services/trust。**  不過，以上針對 Azure AD 實作的標準規則，將會透過簽發者產生 **http://corp.bmcontoso.com/adfs/services/trust** 的權杖。 的權杖，這與網域所需的值不符，因此驗證將會失敗。

### <a name="how-to-enable-support-for-subdomains"></a>如何啟用對於子網域的支援
為了解決此行為，需要更新 Microsoft Online 的 AD FS 信賴憑證者信任。  若要這樣做，您必須設定自訂宣告規則，以使其在建構自訂簽發者值時能夠從使用者的 UPN 尾碼移除任何子網域。

下列宣告將會執行這項操作︰

    c:[Type == "http://schemas.xmlsoap.org/claims/UPN"] => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, "^.*@([^.]+\.)*?(?<domain>([^.]+\.?){2})$", "http://${domain}/adfs/services/trust/"));

[!NOTE]
規則運算式中的最後一個數字會設定您根網域中的父網域數目。 由於此處使用的是 bmcontoso.com，因此必須有兩個父網域。 如果要保留 3 個父網域 (亦即 corp.bmcontoso.com)，則數字就會是 3。 最後可以指出一個範圍，系統一律會進行比對來符合網域數目上限。 "{2,3}" 將比對兩到三個網域 (亦即 bmfabrikam.com 和 corp.bmcontoso.com)。

使用下列步驟來加入自訂宣告，以支援子網域。

1. 開啟 [AD FS 管理]
2. 以滑鼠右鍵按一下 Microsoft Online RP 信任，然後選擇 [編輯宣告規則]
3. 選取第三個宣告規則並取代![編輯宣告](./media/how-to-connect-install-multiple-domains/sub1.png)
4. 取代目前的宣告︰

        c:[Type == "http://schemas.xmlsoap.org/claims/UPN"] => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, ".+@(?<domain>.+)","http://${domain}/adfs/services/trust/"));

       with

        c:[Type == "http://schemas.xmlsoap.org/claims/UPN"] => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, "^.*@([^.]+\.)*?(?<domain>([^.]+\.?){2})$", "http://${domain}/adfs/services/trust/"));

    ![取代宣告](./media/how-to-connect-install-multiple-domains/sub2.png)

5. 按一下 [確定]。  按一下 [套用]。  按一下 [確定]。  關閉 [AD FS 管理]。

## <a name="next-steps"></a>後續步驟
安裝了 Azure AD Connect 之後，您可以 [驗證安裝和指派授權](how-to-connect-post-installation.md)。

深入了解這些在安裝時啟用的功能︰[自動升級](how-to-connect-install-automatic-upgrade.md)、[防止意外刪除](how-to-connect-sync-feature-prevent-accidental-deletes.md)和 [Azure AD Connect Health](how-to-connect-health-sync.md)。

深入了解這些常見主題︰[排程器和如何觸發同步處理](how-to-connect-sync-feature-scheduler.md)。

深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。