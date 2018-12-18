---
title: Azure Active Directory Domain Services：建立群組受管理的服務帳戶 | Microsoft Docs
description: 管理 Azure Active Directory Domain Services 受控網域
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: e6faeddd-ef9e-4e23-84d6-c9b3f7d16567
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/26/2018
ms.author: maheshu
ms.openlocfilehash: 9bfd38b2eba3cab5012e5715ad283b9cb7b84a9b
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39505887"
---
# <a name="create-a-group-managed-service-account-gmsa-on-an-azure-ad-domain-services-managed-domain"></a>在 Azure AD Domain Services 受控網域上建立群組受管理的服務帳戶 (gMSA)
本文說明如何在 Azure AD Domain Services 受控網域上設定受管理的服務帳戶。

## <a name="managed-service-accounts"></a>受管理的服務帳戶
獨立的受管理的服務帳戶 (sMSA) 是會自動管理密碼的受控網域帳戶。 它會簡化服務主體名稱 (SPN) 管理，並為其他系統管理員啟用委派管理。 這種類型的受管理的服務帳戶 (MSA) 是在 Windows Server 2008 R2 和 Windows 7 中引進的。

群組受管理的服務帳戶 (gMSA) 提供與網域上許多伺服器相同的優點。 裝載於伺服器陣列上服務的所有執行個體都需要使用相同的服務主體，相互驗證通訊協定才能運作。 使用 gMSA 作為服務主體時，Windows 作業系統會管理帳戶的密碼，而不會依賴系統管理員。

**更多資訊**
- [群組受管理的服務帳戶概觀](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) \(機器翻譯\)
- [開始使用群組受管理的服務帳戶](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts) \(機器翻譯\)


## <a name="using-service-accounts-in-azure-ad-domain-services"></a>使用 Azure AD Domain Services 中的服務帳戶
Azure AD Domain Services 受控網域會被鎖定並由 Microsoft 管理。 搭配 Azure AD Domain Services 使用服務帳戶時，有幾個重要考量。

### <a name="create-service-accounts-within-custom-organizational-units-ou-on-the-managed-domain"></a>在受控網域上的自訂組織單位 (OU) 中建立服務帳戶
您無法在內建的「AADDC 使用者」或「AADDC 電腦」組織單位中建立服務帳戶。 在受控網域上[建立自訂的 OU](active-directory-ds-admin-guide-create-ou.md)，然後在該自訂的 OU 內建立服務帳戶。

### <a name="the-key-distribution-services-kds-root-key-is-already-pre-created"></a>已經預先建立金鑰發佈服務 (KDS) 根金鑰
金鑰發佈服務 (KDS) 根金鑰會在 Azure AD Domain Services 受控網域上預先建立。 您不需建立 KDS 根金鑰，也沒有執行此動作的權限。 您也無法在受控網域上檢視 KDS 根金鑰。

## <a name="sample---create-a-gmsa-using-powershell"></a>範例：使用 PowerShell 建立 gMSA
下列範例示範如何使用 PowerShell 建立自訂的 OU。 您接著可以使用 ```-Path``` 參數來指定 OU，以在該 OU 內建立 gMSA。

```powershell
# Create a new custom OU on the managed domain
New-ADOrganizationalUnit -Name "MyNewOU" -Path "DC=CONTOSO100,DC=COM"

# Create a service account 'WebFarmSvc' within the custom OU.
New-ADServiceAccount -Name WebFarmSvc  `
-DNSHostName ` WebFarmSvc.contoso100.com  `
-Path "OU=MYNEWOU,DC=CONTOSO100,DC=com"  `
-KerberosEncryptionType AES128, AES256  ` -ManagedPasswordIntervalInDays 30  `
-ServicePrincipalNames http/WebFarmSvc.contoso100.com/contoso100.com, `
http/WebFarmSvc.contoso100.com/contoso100,  `
http/WebFarmSvc/contoso100.com, http/WebFarmSvc/contoso100  `
-PrincipalsAllowedToRetrieveManagedPassword CONTOSO-SERVER$
```

**PowerShell Cmdlet 文件：**
- [New-ADOrganizationalUnit cmdlet](https://docs.microsoft.com/powershell/module/addsadministration/new-adorganizationalunit)
- [New-ADServiceAccount Cmdlet](https://docs.microsoft.com/powershell/module/addsadministration/New-ADServiceAccount)


## <a name="next-steps"></a>後續步驟
- [在受控網域上建立自訂的 OU](active-directory-ds-admin-guide-create-ou.md)
- [群組受管理的服務帳戶概觀](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) \(機器翻譯\)
- [開始使用群組受管理的服務帳戶](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts) \(機器翻譯\)
