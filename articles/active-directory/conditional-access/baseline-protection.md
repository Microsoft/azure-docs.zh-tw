---
title: 何謂 Azure Active Directory 條件式存取中的基準保護？ - 預覽 | Microsoft Docs
description: 了解基準保護如何確保您的 Azure Active Directory 環境中至少會啟用基準層級的安全性。
services: active-directory
keywords: 應用程式的條件式存取, Azure AD 條件式存取, 安全存取公司資源, 條件式存取原則
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.assetid: 8c1d978f-e80b-420e-853a-8bbddc4bcdad
ms.service: active-directory
ms.component: conditional-access
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 08/08/2018
ms.author: markvi
ms.reviewer: nigu
ms.openlocfilehash: ef2b5dd393974ddf700235991b60ec66031e34c2
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47222262"
---
# <a name="what-is-baseline-protection-preview"></a>何謂基準保護 (預覽)？  

去年一年，身分識別攻擊增加了 300%。 為了讓您的環境免於遭受數量持續增加的攻擊，Azure Active Directory (Azure AD) 導入了稱為基準保護的新功能。 基準保護是一組預先定義的[條件式存取原則](../active-directory-conditional-access-azure-portal.md)。 這些原則的目的是要確保所有 Azure AD 版本中至少會啟用基準層級的安全性。 

本文提供 Azure Active Directory 中基準保護的概念性概觀。


 
## <a name="require-mfa-for-admins"></a>管理員需要進行 MFA

有權存取特殊權限帳戶的使用者可以自由存取您的環境。 這些帳戶具有強大權力，您應特別小心處理。 改善特殊權限帳戶保護的常見方法之一，就是在帳戶用於登入時要求更強大的帳戶驗證形式。 在 Azure Active Directory 中，要求多重要素驗證 (MFA)，即可取得比較強大的帳戶驗證。  

[管理員需要進行 MFA] 是一項基準原則，該原則要求下列目錄角色進行 MFA： 

- 全域管理員  

- SharePoint 管理員  

- Exchange 系統管理員  

- 條件式存取系統管理員  

- 安全性系統管理員  


![Azure Active Directory](./media/baseline-protection/01.png)

此基準原則會提供給您排除使用者和群組的選項。 您可以排除一個*[緊急存取系統管理帳戶](../users-groups-roles/directory-emergency-access.md)* 以確保系統不會將您鎖定而讓您無法使用租用戶。


## <a name="enable-a-baseline-policy"></a>啟用基準原則 

雖然基準原則處於預覽狀態，但預設不會啟用。 如果您想要啟用原則，則必須手動啟用。 如果您在預覽階段明確地啟用基準原則，當這項功能正式推出時，基準原則仍會保持作用中狀態。 除了啟用和停用外，正是計劃性的行為變更導致您會有第三個選項可供設定原則的狀態：**未來自動啟用原則**。 選取此選項，您就可以在預覽期間讓原則保持停用，但讓 Microsoft 在這項功能正式推出時自動加以啟用。 如果您現在沒有明確地啟用基準原則，而且未選取 [未來自動啟用原則] 選項，當這項功能正式推出時，原則將保持停用。


**若要啟用基準原則：**  

1. 以全域管理員、安全性系統管理員或條件式存取系統管理員的身分，登入 [Azure 入口網站](https://portal.azure.com)。

2. 在 **Azure 入口網站**的左側導覽列上，按一下 [Azure Active Directory]。

    ![Azure Active Directory](./media/baseline-protection/02.png)

3. 在 [Azure Active Directory] 頁面的 [安全性] 區段中，按一下 [條件式存取]。

    ![條件式存取](./media/baseline-protection/05.png)

4. 在原則清單中，按一下以 [基準原則:] 開頭的原則。 

5. 若要啟用原則，請按一下 [立即使用原則]。

6. 按一下 [檔案] 。 
 
  
 

## <a name="what-you-should-know"></a>您應該知道的事情 

雖然管理自訂條件式存取原則需要 Azure AD Premium 授權，但基準原則適用於所有的 Azure AD 版本。     

基準原則中包含的目錄角色是具有最高特殊權限的 Azure AD 角色。 

如果您有在指令碼中使用的特殊權限帳戶，則應以[受控服務識別 (MSI)](../managed-identities-azure-resources/overview.md) 或[服務主體 (具有憑證)](../../azure-resource-manager/resource-group-authenticate-service-principal.md) 取代這些帳戶。 您暫時可以在基準原則中排除特定使用者帳戶，來解決此問題。 

基準原則適用於舊有驗證流程，例如 POP、IMAP、舊版 Office 桌面用戶端。 




## <a name="next-steps"></a>後續步驟

如需詳細資訊，請參閱

- [可保護身分識別基礎結構的五個步驟](https://docs.microsoft.com/azure/security/azure-ad-secure-steps)

- [什麼是 Azure Active Directory 中的條件式存取？](overview.md) 

