---
title: Azure AD 網域服務︰啟用密碼同步處理 | Microsoft Docs
description: 開始使用 Azure Active Directory Domain Services
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 8731f2b2-661c-4f3d-adba-2c9e06344537
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 11/15/2017
ms.author: maheshu
ms.openlocfilehash: 33cbc65ec5971f8bfe0133e5dca181194cd0bbdd
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46293994"
---
# <a name="enable-password-synchronization-to-azure-active-directory-domain-services"></a>啟用 Azure Active Directory Domain Services 的密碼同步
在先前工作中，您已啟用 Azure Active Directory (Azure AD) 租用戶的 Azure Active Directory Domain Services。 下一項工作是啟用 NT LAN Manager (NTLM) 和 Kerberos 驗證所需的認證雜湊與 Azure AD Domain Services 的同步。 設定認證同步處理後，使用者即可使用他們的公司認證來登入受控網域。

僅限雲端使用者帳戶，與使用 Azure AD Connect 從內部部署目錄同步的使用者帳戶，其所含的步驟不同。

<br>
| **使用者帳戶類型** | **要執行的步驟** |
| --- | --- |
| **已從內部部署目錄同步使用者帳戶** |**&#x2713;** [依照本文的指示](active-directory-ds-getting-started-password-sync-synced-tenant.md#task-5-enable-password-synchronization-to-your-managed-domain-for-user-accounts-synced-with-your-on-premises-ad) | 
| **已在 Azure AD 中建立雲端使用者帳戶** |**&#x2713;** [將僅限雲端使用者帳戶的密碼同步到受控網域](active-directory-ds-getting-started-password-sync.md) |
<br>

> [!TIP]
> **您需要完成這兩組步驟。**
> 如果您的 Azure AD 租用戶同時具有僅限雲端使用者以及來自您內部部署 AD 的使用者，則需要完成這兩組步驟。
>

## <a name="task-5-enable-password-synchronization-to-your-managed-domain-for-user-accounts-synced-with-your-on-premises-ad"></a>工作 5：針對與內部部署 AD 同步的使用者帳戶啟用與受控網域的密碼同步
已同步處理的 Azure AD 租用戶會設定為使用 Azure AD Connect 來與您組織的內部部署目錄同步處理。 根據預設，Azure AD Connect 不會將 NTLM 和 Kerberos 認證雜湊同步到 Azure AD。 若要使用 Azure AD 網域服務，您需要設定 Azure AD Connect 以同步處理 NTLM 和 Kerberos 驗證所需的認證雜湊。 下列步驟能夠將必要的認證雜湊從內部部署目錄同步到 Azure AD 租用戶。

> [!NOTE]
> **如果您的組織具有從您的內部部署目錄同步的使用者帳戶，則您必須啟用 NTLM 與 Kerberos 雜湊的同步，才能使用受控網域。** 同步的使用者帳戶是已在內部部署目錄中建立的帳戶，並會使用 Azure AD Connect 同步到 Azure AD 租用戶。
>
>

### <a name="install-or-update-azure-ad-connect"></a>安裝或更新 Azure AD Connect
在已加入網域的電腦上安裝建議的最新 Azure AD Connect 版本。 如果您目前已設定 Azure AD Connect 的執行個體，則必須加以更新才能使用最新版的 Azure AD Connect。 若要避免已知問題/錯誤可能已經修復，請一律使用最新的 Azure AD Connect 版本。

**[下載 Azure AD Connect](http://www.microsoft.com/download/details.aspx?id=47594)**

建議版本：**1.1.614.0** - 已於 2017 年 9 月 5 日發佈。

> [!WARNING]
> 您「必須」安裝建議的最新 Azure AD Connect 版本，才能將傳統密碼認證 (NTLM 和 Kerberos 驗證所需的認證) 同步處理到 Azure AD 租用戶。 此功能無法在舊版的 Azure AD Connect 中使用，或與舊版 DirSync 工具搭配使用。
>
>

Azure AD Connect 的安裝指示可於下列文章中取得： [開始使用 Azure AD Connect](../active-directory/hybrid/whatis-hybrid-identity.md)

### <a name="enable-synchronization-of-ntlm-and-kerberos-credential-hashes-to-azure-ad"></a>能夠將 NTLM 和 Kerberos 認證雜湊同步處理到 Azure AD
在每個 AD 樹系上執行下列 PowerShell 指令碼。 指令碼可讓所有內部部署使用者的 NTLM 與 Kerberos 密碼雜湊同步處理至 Azure AD 租用戶。 指令碼也會在 Azure AD Connect 中起始完整的同步處理。

```powershell
$adConnector = "<CASE SENSITIVE AD CONNECTOR NAME>"  
$azureadConnector = "<CASE SENSITIVE AZURE AD CONNECTOR NAME>"  
Import-Module adsync  
$c = Get-ADSyncConnector -Name $adConnector  
$p = New-Object Microsoft.IdentityManagement.PowerShell.ObjectModel.ConfigurationParameter "Microsoft.Synchronize.ForceFullPasswordSync", String, ConnectorGlobal, $null, $null, $null
$p.Value = 1  
$c.GlobalParameters.Remove($p.Name)  
$c.GlobalParameters.Add($p)  
$c = Add-ADSyncConnector -Connector $c  
Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $azureadConnector -Enable $false   
Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $azureadConnector -Enable $true  
```

視目錄的大小而定 (使用者、群組等的數目)，將認證雜湊同步處理到 Azure AD 需要花一點時間。 將認證雜湊同步處理到 Azure AD 之後，密碼短時間內就能在 Azure Actuve Directory Domain Services 的受控網域上使用。

<br>

## <a name="related-content"></a>相關內容
* [為僅限雲端的 Azure AD 目錄啟用 AAD 網域服務的密碼同步處理](active-directory-ds-getting-started-password-sync.md)
* [Administer an Azure AD Domain Services managed domain (管理 Azure AD 網域服務受控網域)](active-directory-ds-admin-guide-administer-domain.md)
* [將 Windows 虛擬機器加入 Azure AD 網域服務受控網域](active-directory-ds-admin-guide-join-windows-vm.md)
* [將 Red Hat Enterprise Linux 虛擬機器加入 Azure AD 網域服務受控網域](active-directory-ds-admin-guide-join-rhel-linux-vm.md)
