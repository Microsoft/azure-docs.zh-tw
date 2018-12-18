---
title: 對許多應用程式授與 Azure 金鑰保存庫的存取權限 | Microsoft Docs
description: 了解如何對許多應用程式授與金鑰保存庫的存取權限
services: key-vault
documentationcenter: ''
author: amitbapat
manager: mbaldwin
tags: azure-resource-manager
ms.assetid: 785d4e40-fb7b-485a-8cbc-d9c8c87708e6
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/01/2016
ms.author: ambapat
ms.openlocfilehash: 14e6e8bb723eb236f8fb315454b8697a3bd947ef
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43286422"
---
# <a name="grant-permission-to-many-applications-to-access-a-key-vault"></a>對許多應用程式授與金鑰保存庫的存取權限

## <a name="q-i-have-several-applications-that-need-to-access-a-key-vault-how-can-i-give-these-applications-up-to-1024-access-to-key-vault"></a>問：我有數個需要存取 Key Vault 的應用程式，如何讓這些應用程式 (最多 1024 個) 存取 Key Vault？

Key Vault 存取控制原則最多支援 1024 個項目。 不過，您可以建立 Azure Active Directory 安全性群組。 將所有相關聯的服務主體新增至這個安全性群組，然後對 Key Vault 授與此安全性群組的存取權。

必要條件如下︰
* [安裝 Azure Active Directory V2 PowerShell 模組](https://www.powershellgallery.com/packages/AzureAD)。
* [安裝 Azure PowerShell](/powershell/azure/overview)。
* 若要執行下列命令，您需要在 Azure Active Directory 租用戶中建立/編輯群組的權限。 如果您沒有權限，則可能需要連絡 Azure Active Directory 系統管理員。

現在在 PowerShell 中執行下列命令。

```powershell
# Connect to Azure AD 
Connect-AzureAD 
 
# Create Azure Active Directory Security Group 
$aadGroup = New-AzureADGroup -Description "Contoso App Group" -DisplayName "ContosoAppGroup" -MailEnabled 0 -MailNickName none -SecurityEnabled 1 
 
# Find and add your applications (ServicePrincipal ObjectID) as members to this group 
$spn = Get-AzureADServicePrincipal –SearchString "ContosoApp1" 
Add-AzureADGroupMember –ObjectId $aadGroup.ObjectId -RefObjectId $spn.ObjectId 
 
# You can add several members to this group, in this fashion. 
 
# Set the Key Vault ACLs 
Set-AzureRmKeyVaultAccessPolicy –VaultName ContosoVault –ObjectId $aadGroup.ObjectId -PermissionsToKeys all –PermissionsToSecrets all –PermissionsToCertificates all 
 
# Of course you can adjust the permissions as required 
```

如果您需要對一組應用程式授與一組不同的權限，請為這些應用程式另外建立一個 Azure Active Directory 安全性群組。

## <a name="next-steps"></a>後續步驟

深入了解如何[保護您的金鑰保存庫](key-vault-secure-your-key-vault.md)。
