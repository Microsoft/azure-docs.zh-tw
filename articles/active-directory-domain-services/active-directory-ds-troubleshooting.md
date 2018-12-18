---
title: Azure Active Directory Domain Services：疑難排解指南 | Microsoft Docs
description: Azure AD 網域服務的疑難排解指南
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 4bc8c604-f57c-4f28-9dac-8b9164a0cf0b
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/08/2018
ms.author: maheshu
ms.openlocfilehash: fa4d40786fdc61183c119a17b6e36bd853a6fd64
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42140895"
---
# <a name="azure-ad-domain-services---troubleshooting-guide"></a>Azure AD 網域服務 - 疑難排解指南
這篇文章提供設定或管理 Azure Active Directory (AD) 網域服務時，可能會遇到的問題之疑難排解提示。

## <a name="you-cannot-enable-azure-ad-domain-services-for-your-azure-ad-directory"></a>您無法為 Azure AD 目錄啟用 Azure AD 網域服務
若您在嘗試為目錄啟用 Azure AD Domain Services 時，本節可協助您針對錯誤進行疑難排解。

挑選對應至您所遇到之錯誤訊息的疑難排解步驟。

| **錯誤訊息** | **解決方案** |
| --- |:--- |
| *名稱 contoso100.com 使用於此網路上。指定未使用的名稱。* |[虛擬網路中的網域名稱衝突](active-directory-ds-troubleshooting.md#domain-name-conflict) |
| *無法在此 Azure AD 租用戶中啟用網域服務。對於名為「Azure AD 網域服務同步處理」的應用程式，此服務沒有足夠的權限。刪除名為「Azure AD 網域服務同步處理」的應用程式，然後嘗試為 Azure AD 租用戶啟用網域服務。* |[網域服務沒有「Azure AD 網域服務同步處理」應用程式的足夠權限。](active-directory-ds-troubleshooting.md#inadequate-permissions) |
| *無法在此 Azure AD 租用戶中啟用網域服務。Azure AD 租用戶中的網域服務應用程式沒有啟用網域服務所需的權限。刪除應用程式識別碼為 d87dcbc6-a371-462e-88e3-28ad15ec4e64 的應用程式，然後嘗試為 Azure AD 租用戶啟用網域服務。* |[未在您的租用戶中正確設定網域服務應用程式](active-directory-ds-troubleshooting.md#invalid-configuration) |
| *無法在此 Azure AD 租用戶中啟用網域服務。您的 Azure AD 租用戶已停用 Microsoft Azure AD 應用程式。啟用應用程式識別碼為 00000002-0000-0000-c000-000000000000 的應用程式，然後嘗試為 Azure AD 租用戶啟用網域服務。* |[您的 Azure AD 租用戶已停用 Microsoft Graph 應用程式](active-directory-ds-troubleshooting.md#microsoft-graph-disabled) |

### <a name="domain-name-conflict"></a>網域名稱衝突
**錯誤訊息：**

*名稱 contoso100.com 使用於此網路上。指定未使用的名稱。*

**補救：**

請確定現有網域的名稱並未與該虛擬網路上可用的網域名稱相同。 例如，假設您有名為 'contoso.com' 的網域已可用於選取的虛擬網路。 接著，您可嘗試在該虛擬網路上啟用具有相同網域名稱 (即 'contoso.com') 的 Azure AD 網域服務受控網域。 您在嘗試啟用 Azure AD 網域服務時發生錯誤。

這個錯誤是因為名稱與該虛擬網路上的網域名稱衝突。 在此情況下，您必須使用不同的名稱來設定 Azure AD 網域服務受控網域。 或者，您可以解除佈建現有的網域，然後繼續啟用 Azure AD 網域服務。

### <a name="inadequate-permissions"></a>權限不足
**錯誤訊息：**

*無法在此 Azure AD 租用戶中啟用網域服務。對於名為「Azure AD 網域服務同步處理」的應用程式，此服務沒有足夠的權限。刪除名為「Azure AD 網域服務同步處理」的應用程式，然後嘗試為 Azure AD 租用戶啟用網域服務。*

**補救：**

查看 Azure AD 目錄中是否有名稱為「Azure AD 網域服務同步處理」的應用程式。 如果此應用程式存在，請將它刪除，然後再重新啟用 Azure AD 網域服務。

執行下列步驟，以檢查應用程式是否存在並將它刪除 (如果應用程式存在的話)：

1. 在 [Azure 入口網站](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps/menuId/)中瀏覽至 Azure AD 目錄的 [應用程式] 區段。
2. 選取 [顯示] 下拉式清單中的 [所有應用程式]。 選取 [應用程式狀態] 下拉式清單中的 [任何]。 選取 [應用程式可見度] 下拉式清單中的 [任何]。
3. 在搜尋方塊中輸入 **Azure AD Domain Services 同步**。 如果該應用程式存在，請對它按一下，然後按一下工具列中的 [刪除] 按鈕以將它刪除。
4. 刪除此應用程式後，請嘗試再次啟用 Azure AD 網域服務。

### <a name="invalid-configuration"></a>無效的組態
**錯誤訊息：**

*無法在此 Azure AD 租用戶中啟用網域服務。Azure AD 租用戶中的網域服務應用程式沒有啟用網域服務所需的權限。刪除應用程式識別碼為 d87dcbc6-a371-462e-88e3-28ad15ec4e64 的應用程式，然後嘗試為 Azure AD 租用戶啟用網域服務。*

**補救：**

請查看 Azure AD 目錄中是否有名稱為 'AzureActiveDirectoryDomainControllerServices' (應用程式識別碼為 d87dcbc6-a371-462e-88e3-28ad15ec4e64) 的應用程式。 如果此應用程式存在，您必須將它刪除，然後再重新啟用 Azure AD 網域服務。

使用下列 PowerShell 指令碼來尋找此應用程式並加以刪除。

> [!NOTE]
> 此指令碼會使用 **Azure AD PowerShell 第 2 版** Cmdlet。 如需所有可用 Cmdlet 的完整清單及下載此模組，請參閱 [AzureAD PowerShell 參考文件](https://msdn.microsoft.com/library/azure/mt757189.aspx)。
>
>

```powershell
$InformationPreference = "Continue"
$WarningPreference = "Continue"

$aadDsSp = Get-AzureADServicePrincipal -Filter "AppId eq 'd87dcbc6-a371-462e-88e3-28ad15ec4e64'" -ErrorAction Ignore
if ($aadDsSp -ne $null)
{
    Write-Information "Found Azure AD Domain Services application. Deleting it ..."
    Remove-AzureADServicePrincipal -ObjectId $aadDsSp.ObjectId
    Write-Information "Deleted the Azure AD Domain Services application."
}

$identifierUri = "https://sync.aaddc.activedirectory.windowsazure.com"
$appFilter = "IdentifierUris eq '" + $identifierUri + "'"
$app = Get-AzureADApplication -Filter $appFilter
if ($app -ne $null)
{
    Write-Information "Found Azure AD Domain Services Sync application. Deleting it ..."
    Remove-AzureADApplication -ObjectId $app.ObjectId
    Write-Information "Deleted the Azure AD Domain Services Sync application."
}

$spFilter = "ServicePrincipalNames eq '" + $identifierUri + "'"
$sp = Get-AzureADServicePrincipal -Filter $spFilter
if ($sp -ne $null)
{
    Write-Information "Found Azure AD Domain Services Sync service principal. Deleting it ..."
    Remove-AzureADServicePrincipal -ObjectId $sp.ObjectId
    Write-Information "Deleted the Azure AD Domain Services Sync service principal."
}
```
<br>

### <a name="microsoft-graph-disabled"></a>Microsoft Graph 已停用
**錯誤訊息：**

無法在此 Azure AD 租用戶中啟用網域服務。 您的 Azure AD 租用戶已停用 Microsoft Azure AD 應用程式。 啟用應用程式識別碼為 00000002-0000-0000-c000-000000000000 的應用程式，然後嘗試為 Azure AD 租用戶啟用網域服務。

**補救：**

請查看您是否已停用識別碼為 00000002-0000-0000-c000-000000000000 的應用程式。 此應用程式是 Microsoft Azure AD 應用程式，可提供 Azure AD 租用戶的圖形 API 存取權。 Azure AD 網域服務需要啟用此應用程式，才能將 Azure AD 租用戶同步處理至受控網域。

若要解決此錯誤，請啟用此應用程式，然後嘗試為 Azure AD 租用戶啟用網域服務。


## <a name="users-are-unable-to-sign-in-to-the-azure-ad-domain-services-managed-domain"></a>Azure Active Directory Domain Services 受控的網域
如果 Azure AD 租用戶中有一或多個使用者無法登入新建立的受控網域，請執行下列疑難排解步驟：

* **使用 UPN 格式進行登入：** 嘗試使用 UPN 格式 (例如，joeuser@contoso.com) 進行登入，而非使用 SAMAccountName 格式 ('CONTOSO\joeuser')。 對於 UPN 前置詞過長或與受控網域上其他使用者相同的使用者，可能會自動產生 SAMAccountName。 UPN 格式保證是 Azure AD 租用戶內唯一的格式。

> [!NOTE]
> 建議使用 UPN 格式來登入 Azure AD 網域服務受控網域。
>
>

* 確定您已根據《入門指南》中所述的步驟來 [啟用密碼同步處理](active-directory-ds-getting-started-password-sync.md) 。
* **外部帳戶** 確定受影響的使用者帳戶不是 Azure AD 租用戶中的外部帳戶。 外部帳戶的範例包括 Microsoft 帳戶 (例如 joe@live.com) 或來自外部 Azure AD 目錄的使用者帳戶。 因為 Azure Active Directory Domain Services 沒有這類使用者帳戶的認證，這些使用者會無法登入受控網域。
* **同步處理的帳戶：** 如果受影響的使用者帳戶會從內部部署目錄同步處理，請確認：

  * 您已部署或更新為 [Azure AD Connect 的最新版本](https://www.microsoft.com/download/details.aspx?id=47594)。
  * 您已設定 Azure AD Connect 以 [執行完整同步處理](active-directory-ds-getting-started-password-sync.md)。
  * 根據您的目錄大小，可能需要一些時間，使用者帳戶和認證雜湊才可在 Azure AD 網域服務中提供使用。 請確定您已等候足夠的時間後，再重試驗證。
  * 如果在您驗證上述步驟之後問題仍持續發生，請嘗試重新啟動 Microsoft Azure AD 同步服務。 在您的同步電腦上，啟動命令提示字元，然後執行下列命令：

    1. net stop 'Microsoft Azure AD Sync'
    2. net start 'Microsoft Azure AD Sync'
* **僅限雲端帳戶**：如果受影響的使用者帳戶是僅限雲端的使用者帳戶，請確定在您啟用 Azure AD 網域服務之後，使用者已變更其密碼。 這個步驟會導致產生 Azure AD 網域服務所需的認證雜湊。

## <a name="there-are-one-or-more-alerts-on-your-managed-domain"></a>受控網域有一或多個警示

瀏覽[疑難排解警示](active-directory-ds-troubleshoot-alerts.md)一文，了解如何解決受控網域的警示。

## <a name="users-removed-from-your-azure-ad-tenant-are-not-removed-from-your-managed-domain"></a>不會從受控網域中移除從 Azure AD 租用戶移除的使用者
Azure AD 會防止您意外刪除使用者物件。 當您從您的 Azure AD 租用戶刪除使用者帳戶時，對應的使用者物件會移至資源回收筒。 此刪除作業同步處理至受控網域時，會導致對應的使用者帳戶被標記為已停用。 這項功能可協助您稍後復原或取消刪除使用者帳戶。

即使您在 Azure AD 目錄中使用相同的 UPN 重新建立使用者帳戶，使用者帳戶在您的受控網域中仍會維持停用狀態。 若要從受控網域移除使用者帳戶，請從您的 Azure AD 租用戶將它強制刪除。

若要完全從受控網域移除使用者帳戶，請從您的 Azure AD 租用戶中永久刪除使用者。 請使用 `Remove-MsolUser` PowerShell Cmdlet 搭配 `-RemoveFromRecycleBin` 選項，如這篇 [MSDN 文章](https://msdn.microsoft.com/library/azure/dn194132.aspx)所述。


## <a name="contact-us"></a>與我們連絡
請連絡 Azure Active Directory Domain Services 產品小組， [分享意見或尋求支援](active-directory-ds-contact-us.md)。
