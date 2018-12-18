---
title: 在 Azure AD Domain Services 中設定安全的 LDAP (LDAPS) | Microsoft Docs
description: 針對 Azure Active Directory Domain Services 受控網域設定安全的 LDAP (LDAPS)
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: c6da94b6-4328-4230-801a-4b646055d4d7
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/25/2018
ms.author: maheshu
ms.openlocfilehash: 22c97da35416ba1ff593dfa5e41f557ea2ab1cc0
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47182241"
---
# <a name="configure-secure-ldap-ldaps-for-an-azure-ad-domain-services-managed-domain"></a>針對 Azure AD 網域服務受控網域設定安全的 LDAP (LDAPS)
本文說明如何為 Azure Active Directory Domain Services 受控網域啟用安全的輕量型目錄存取通訊協定 (LDAPS)。 安全的 LDAP 亦稱為「透過安全通訊端層 (SSL)/傳輸層安全性 (TLS) 的輕量型目錄存取通訊協定 (LDAP)」。

[!INCLUDE [active-directory-ds-prerequisites.md](../../includes/active-directory-ds-prerequisites.md)]

## <a name="before-you-begin"></a>開始之前
若要執行本文中所列的工作，您需要︰

1. 有效的 **Azure 訂用帳戶**。
2. **Azure AD 目錄** - 與內部部署目錄或僅限雲端的目錄同步處理。
3. **Azure AD 網域服務** 必須已針對 Azure AD 目錄啟用。 如果還沒有啟用，請按照 [入門指南](active-directory-ds-getting-started.md)所述的所有工作來進行。
4. **用來啟用安全 LDAP 的憑證**。

   * **建議** - 從受信任的公共憑證授權單位取得憑證。 這是更安全的組態選項。
   * 或者，您也可以選擇 [建立自我簽署憑證](#task-1---obtain-a-certificate-for-secure-ldap) ，如本文稍後所示。

<br>

### <a name="requirements-for-the-secure-ldap-certificate"></a>安全 LDAP 憑證的需求
請先根據下列準則取得有效的憑證，再啟用安全的 LDAP。 如果您嘗試使用無效/不正確的憑證來為受控網域啟用安全的 LDAP，您會遭遇失敗。

1. **信任的簽發者** - 憑證必須由使用安全 LDAP 連線到網域的電腦，所信任的授權單位加以發行。 此授權單位可能是受這些電腦信任的公用憑證授權單位 (CA) 或企業 CA。
2. **存留期** - 憑證必須至少在接下來的 3 至 6 個月內都要保持有效。 當憑證過期時，受控網域的安全 LDAP 存取會中斷。
3. **主體名稱** - 在受控網域中，憑證的主體名稱必須是萬用字元。 比方說，如果您的網域名稱為 'contoso100.com'，則憑證的主體名稱必須是 '*.contoso100.com'。 設定此萬用字元名稱的 DNS 名稱 (主體替代名稱)。
4. **金鑰使用方法** - 必須將憑證設定為下列用途 - 數位簽章與金鑰編密。
5. **憑證目的** - 憑證必須有效可進行 SSL 伺服器驗證。

<br>

## <a name="task-1---obtain-a-certificate-for-secure-ldap"></a>工作 1 - 取得安全 LDAP 的憑證
第一項工作牽涉到取得憑證，該憑證用於對受控網域進行安全的 LDAP 存取。 您有兩個選擇：

* 從公用 CA 或企業 CA 取得憑證。
* 建立自我簽署憑證。

> [!NOTE]
> 需要使用安全的 LDAP 連線到受控網域的用戶端電腦必須信任安全 LDAPS 憑證的簽發者。
>

### <a name="option-a-recommended---obtain-a-secure-ldap-certificate-from-a-certification-authority"></a>選項 A (建議選項) - 從憑證授權單位取得安全的 LDAP 憑證
如果您的組織從公用 CA 取得其憑證，請從該公用 CA 取得安全的 LDAP 憑證。 如果您部署企業 CA，請從企業 CA 取得安全 LDAP 的憑證。

> [!TIP]
> **針對網域尾碼為 '.onmicrosoft.com' 的受控網域，請使用自我簽署的憑證。**
> 如果受控網域的 DNS 網域名稱結尾是 '.onmicrosoft.com'，您就無法從公開憑證授權單位取得安全 LDAP 憑證。 由於 Microsoft 擁有 'onmicrosoft.com' 網域，因此公開憑證授權單位會拒絕對您簽發具有此尾碼之網域的安全 LDAP 憑證。 在此情況下，請建立自我簽署的憑證，然後使用該憑證來設定安全 LDAP。
>

請確定您從公開憑證授權單位取得的憑證符合[安全 LDAP 憑證的需求](#requirements-for-the-secure-ldap-certificate)中所述的需求。


### <a name="option-b---create-a-self-signed-certificate-for-secure-ldap"></a>選項 B - 為安全的 LDAP 建立自我簽署的憑證
如果您不希望使用公開憑證授權單位的憑證，可以選擇建立安全 LDAP 的自我簽署憑證。 如果受控網域的 DNS 網域名稱結尾是 '.onmicrosoft.com'，請選擇此選項。

**使用 PowerShell 建立自我簽署憑證**

在您的 Windows 電腦上以 **系統管理員** 的身分開啟新的 PowerShell 視窗，並輸入下列命令以建立新的自我簽署憑證。

```powershell
$lifetime=Get-Date
New-SelfSignedCertificate -Subject contoso100.com `
  -NotAfter $lifetime.AddDays(365) -KeyUsage DigitalSignature, KeyEncipherment `
  -Type SSLServerAuthentication -DnsName *.contoso100.com
```

在上述範例中，將 '*.contoso100.com' 替換為受控網域的 DNS 網域名稱。例如，如果您建立名為 'contoso100.onmicrosoft.com' 的受控網域，請將上述指令碼中的 '*.contoso100.com' 替換成 '*.contoso100.onmicrosoft.com'。

![選取 Azure AD 目錄](./media/active-directory-domain-services-admin-guide/secure-ldap-powershell-create-self-signed-cert.png)

新建立的自我簽署憑證會放在本機電腦的憑證存放區中。


## <a name="next-step"></a>後續步驟
[工作 2 - 將安全 LDAP 憑證匯出到 .PFX 檔案](active-directory-ds-admin-guide-configure-secure-ldap-export-pfx.md)
