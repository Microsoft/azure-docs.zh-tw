---
title: 針對已加入混合式 Azure Active Directory 的 Windows 10 和 Windows Server 2016 裝置進行疑難排解 | Microsoft Docs
description: 針對已加入混合式 Azure Active Directory 的 Windows 10 和 Windows Server 2016 裝置進行疑難排解。
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
ms.assetid: cdc25576-37f2-4afb-a786-f59ba4c284c2
ms.service: active-directory
ms.component: devices
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/08/2017
ms.author: markvi
ms.reviewer: jairoc
ms.openlocfilehash: 1d96c1e8adee55127a50b2d7c374418c22bfec4c
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43050560"
---
# <a name="troubleshooting-hybrid-azure-active-directory-joined-windows-10-and-windows-server-2016-devices"></a>針對已加入混合式 Azure Active Directory 的 Windows 10 和 Windows Server 2016 裝置進行疑難排解 

本文適用於下列用戶端：

-   Windows 10
-   Windows Server 2016

至於其他 Windows 用戶端，請參閱[針對已加入混合式 Azure Active Directory 的下層裝置進行疑難排解](troubleshoot-hybrid-join-windows-legacy.md)。

本文章假設您[設定已加入混合式 Azure Active Directory 的裝置](hybrid-azuread-join-plan.md)來支援下列案例：

- 裝置型條件式存取

- [設定的企業漫遊](../active-directory-windows-enterprise-state-roaming-overview.md)

- [Windows Hello 企業版](../active-directory-azureadjoin-passport-deployment.md)


本文件提供有關如何解決潛在問題的疑難排解指引。 


對於 Windows 10 和 Windows Server 2016，混合式 Azure Active Directory 會加入對 Windows 10 2015 年 11 月更新 (含) 以上版本的支援。 建議使用年度更新版。

## <a name="step-1-retrieve-the-join-status"></a>步驟 1：擷取加入狀態 

**若要擷取加入狀態：**

1. 以系統管理員身分開啟命令提示字元

2. 輸入 **dsregcmd /status**



    +----------------------------------------------------------------------+
    | Device State                                                         |  +----------------------------------------------------------------------+
    
        AzureAdJoined: YES
     EnterpriseJoined: NO DeviceId: 5820fbe9-60c8-43b0-bb11-44aee233e4e7 Thumbprint: B753A6679CE720451921302CA873794D94C6204A KeyContainerId: bae6a60b-1d2f-4d2a-a298-33385f6d05e9 KeyProvider: Microsoft Platform Crypto Provider TpmProtected: YES KeySignTest: : MUST Run elevated to test.
                  Idp: login.windows.net TenantId: 72b988bf-86f1-41af-91ab-2d7cd011db47 TenantName: Contoso AuthCodeUrl: https://login.microsoftonline.com/msitsupp.microsoft.com/oauth2/authorize AccessTokenUrl: https://login.microsoftonline.com/msitsupp.microsoft.com/oauth2/token MdmUrl: https://enrollment.manage-beta.microsoft.com/EnrollmentServer/Discovery.svc MdmTouUrl: https://portal.manage-beta.microsoft.com/TermsOfUse.aspx dmComplianceUrl: https://portal.manage-beta.microsoft.com/?portalAction=Compliance SettingsUrl: eyJVcmlzIjpbImh0dHBzOi8va2FpbGFuaS5vbmUubWljcm9zb2Z0LmNvbS8iLCJodHRwczovL2thaWxhbmkxLm9uZS5taWNyb3NvZnQuY29tLyJdfQ== JoinSrvVersion: 1.0 JoinSrvUrl: https://enterpriseregistration.windows.net/EnrollmentServer/device/ JoinSrvId: urn:ms-drs:enterpriseregistration.windows.net KeySrvVersion: 1.0 KeySrvUrl: https://enterpriseregistration.windows.net/EnrollmentServer/key/ KeySrvId: urn:ms-drs:enterpriseregistration.windows.net DomainJoined: YES DomainName: CONTOSO
    
    +----------------------------------------------------------------------+
    | User State                                                           |  +----------------------------------------------------------------------+
    
                 NgcSet: YES
               NgcKeyId: {C7A9AEDC-780E-4FDA-B200-1AE15561A46B}
        WorkplaceJoined: NO
          WamDefaultSet: YES
    WamDefaultAuthority: organizations         WamDefaultId: https://login.microsoft.com       WamDefaultGUID: {B16898C6-A148-4967-9171-64D755DA8520} (AzureAd)           AzureAdPrt: YES



## <a name="step-2-evaluate-the-join-status"></a>步驟 2：評估加入狀態 

請檢閱下列欄位，並確定它們具有預期的值：

### <a name="azureadjoined--yes"></a>AzureAdJoined : YES  

此欄位指出裝置是否已加入 Azure AD。 如果值為 **NO**，則尚未完成加入 Azure AD。 

**可能的原因：**

- 驗證要加入的電腦失敗。

- 組織中有電腦探索不到的 HTTP Proxy。

- 電腦無法連線到 Azure AD 來進行驗證，或無法連線到 Azure DRS 來進行註冊

- 電腦可能不在組織的內部網路上，或不在可直接看到內部部署 AD 網域控制站的 VPN 上。

- 如果電腦有 TPM，它可能是處於不正常狀態。

- 在本文件稍早提到的服務中可能有設定錯誤的情況，您將必須重新確認。 常見範例包括：

    - 您的同盟伺服器未啟用 WS-Trust 端點

    - 您的同盟伺服器不允許來自您網路中使用「整合式 Windows 驗證」之電腦的輸入驗證。

    - 沒有「服務連接點」物件指向電腦所屬 AD 樹系之 Azure AD 中已驗證的網域名稱

---

### <a name="domainjoined--yes"></a>DomainJoined : YES  

此欄位指出裝置是否已加入內部部署 Active Directory。 如果值為 **NO**，則裝置無法執行混合式 Azure AD 加入。  

---

### <a name="workplacejoined--no"></a>WorkplaceJoined : NO  

此欄位指出裝置是否已向 Azure AD 註冊為個人裝置 (標示為「已加入工作場所」)。 如果已加入網域的電腦同時加入混合式 Azure AD，此值應為 **NO**。 如果值為 **YES**，則在完成混合式 Azure AD 加入之前已新增工作或學校帳戶。 在此情況下，使用年度更新版的 Windows 10 (1607) 時會忽略該帳戶。

---

### <a name="wamdefaultset--yes-and-azureadprt--yes"></a>WamDefaultSet : YES 和 AzureADPrt : YES
  
這些欄位指出使用者在登入裝置時，是否已順利向 Azure AD 進行驗證。 如果值為 **NO**，可能是因為：

- 註冊時，與裝置關聯之 TPM 中的儲存體金鑰 (STK) 無效 (請在提高權限執行的情況下，檢查 KeySignTest)。

- 替代登入識別碼

- 找不到 HTTP Proxy

## <a name="next-steps"></a>後續步驟

如有問題，請參閱[裝置管理常見問題集](faq.md) 