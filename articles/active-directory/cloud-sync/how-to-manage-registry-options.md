---
title: Azure AD Connect 雲端布建代理程式：管理登錄選項 |Microsoft Docs
description: 本文說明如何在 Azure AD Connect 雲端布建代理程式中管理登錄選項。
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/11/2020
ms.subservice: hybrid
ms.reviewer: chmutali
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: e4cdda52271bc7b9e9d854e0af181e2c8f22ad9a
ms.sourcegitcommit: 8a74ab1beba4522367aef8cb39c92c1147d5ec13
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/20/2021
ms.locfileid: "98613131"
---
# <a name="manage-agent-registry-options"></a>管理代理程式登錄選項

本節說明您可以設定的登錄選項，以控制 Azure AD Connect 布建代理程式的執行時間處理行為。 

## <a name="configure-ldap-connection-timeout"></a>設定 LDAP 連接逾時
在設定的 Active Directory 網域控制站上執行 LDAP 作業時，布建代理程式預設會使用預設連接逾時值30秒。 如果您的網域控制站需要更多時間來回應，則您可能會在代理程式記錄檔中看到下列錯誤訊息： 

`
System.DirectoryServices.Protocols.LdapException: The operation was aborted because the client side timeout limit was exceeded.
`

如果搜尋屬性未建立索引，LDAP 搜尋作業可能需要較長的時間。 第一個步驟是，如果您收到上述錯誤，請先檢查搜尋/查閱屬性是否已 [編制索引](https://docs.microsoft.com/windows/win32/ad/indexed-attributes)。 如果搜尋屬性已編制索引，但錯誤持續發生，您可以使用下列步驟來增加 LDAP 連接逾時： 

1. 在執行 Azure AD Connect 布建代理程式的 Windows server 上，以系統管理員身分登入。
1. 使用 [ *執行* ] 功能表項目開啟 [登錄編輯程式] ( # A0)  
1. 找出金鑰資料夾 **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Azure AD Connect Agents\Azure AD Connect Provisioning Agent**
1. 以滑鼠右鍵按一下並選取 [新增-> 字串值]
1. 提供名稱： `LdapConnectionTimeoutInMilliseconds`
1. 按兩下 **值名稱** ，並以毫秒為單位輸入值資料 `60000` 。
    > [!div class="mx-imgBorder"]
    > ![LDAP 連接逾時](media/how-to-manage-registry-options/ldap-connection-timeout.png)
1. 從 [ *服務* ] 主控台重新開機 Azure AD Connect 布建服務。
1. 如果您已部署多個布建代理程式，請將此登錄變更套用至所有代理程式的一致性。 

## <a name="configure-referral-chasing"></a>設定參考追蹤
根據預設，Azure AD Connect 布建代理程式不會執行 [參考](https://docs.microsoft.com/windows/win32/ad/referrals)。 您可能會想要啟用參考追蹤，以支援特定的 HR 輸入布建案例，例如： 
* 檢查跨多個網域的 UPN 唯一性
* 解析跨網域管理員參考

使用下列步驟來開啟參考追蹤：

1. 在執行 Azure AD Connect 布建代理程式的 Windows server 上，以系統管理員身分登入。
1. 使用 [ *執行* ] 功能表項目開啟 [登錄編輯程式] ( # A0)  
1. 找出金鑰資料夾 **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Azure AD Connect Agents\Azure AD Connect Provisioning Agent**
1. 以滑鼠右鍵按一下並選取 [新增-> 字串值]
1. 提供名稱： `ReferralChasingOptions`
1. 按兩下 **值名稱** ，然後輸入值資料做為 `96` 。 這個值會對應至的常數值 `ReferralChasingOptions.All` ，並指定子樹和基底層級的參考會接著代理程式。 
    > [!div class="mx-imgBorder"]
    > ![參考追蹤](media/how-to-manage-registry-options/referral-chasing.png)
1. 從 [ *服務* ] 主控台重新開機 Azure AD Connect 布建服務。
1. 如果您已部署多個布建代理程式，請將此登錄變更套用至所有代理程式的一致性。

## <a name="skip-gmsa-configuration"></a>略過 GMSA 設定
使用代理程式版本1.1.281.0 版 + 時，根據預設，當您執行 [代理程式設定] 時，系統會提示您設定 [群組受管理的服務帳戶 (GMSA) ](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)。 Wizard 的 GMSA 設定會在執行時間用於所有同步處理和布建作業。 

如果您要從舊版的代理程式進行升級，並設定自訂的服務帳戶，且該帳戶具有您 Active Directory 拓撲專屬的委派 OU 層級許可權，您可能會想要略過/延期 GMSA 設定並規劃這項變更。 

> [!NOTE]
> 本指南特別適用于已設定 HR (Workday/SuccessFactors) 輸入布建與1.1.281.0 版之前的代理程式版本，並設定代理程式作業的自訂服務帳戶的客戶。 在長時間執行時，建議您改用 GMSA 作為最佳作法。  

在此案例中，您仍然可以升級代理程式二進位檔，並使用下列步驟來略過 GMSA 設定： 

1. 在執行 Azure AD Connect 布建代理程式的 Windows server 上，以系統管理員身分登入。
1. 執行代理程式安裝程式，以安裝新的代理程式二進位檔。 關閉安裝成功後自動開啟的 [代理程式設定]。 
1. 使用 [ *執行* ] 功能表項目開啟 [登錄編輯程式] ( # A0)  
1. 找出金鑰資料夾 **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Azure AD Connect Agents\Azure AD Connect Provisioning Agent**
1. 以滑鼠右鍵按一下並選取 [新增-> DWORD 值]
1. 提供名稱： `UseCredentials`
1. 按兩下 **值名稱** ，然後輸入值資料做為 `1` 。  
    > [!div class="mx-imgBorder"]
    > ![使用認證](media/how-to-manage-registry-options/use-credentials.png)
1. 從 [ *服務* ] 主控台重新開機 Azure AD Connect 布建服務。
1. 如果您已部署多個布建代理程式，請將此登錄變更套用至所有代理程式的一致性。
1. 從桌面短暫剪下，執行「代理程式設定向導」。 Wizard 將略過 GMSA 設定。 


> [!NOTE]
> 您可以藉由啟用 [詳細資訊記錄](how-to-troubleshoot.md#log-files)來確認已設定登錄選項。 代理程式啟動期間發出的記錄會顯示從登錄中挑選的設定值。 

## <a name="next-steps"></a>後續步驟 

- [什麼是佈建？](what-is-provisioning.md)
- [什麼是雲端同步 Azure AD Connect？](what-is-cloud-sync.md)

