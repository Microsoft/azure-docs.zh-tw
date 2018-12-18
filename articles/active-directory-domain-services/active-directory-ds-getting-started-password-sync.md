---
title: Azure Active Directory Domain Services︰啟用密碼雜湊同步 | Microsoft Docs
description: 開始使用 Azure Active Directory Domain Services
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 5a32a0df-a3ca-4ebe-b980-91f58f8030fc
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/02/2018
ms.author: maheshu
ms.openlocfilehash: 0f7023b60ef3678c284fe05bff7be73674595512
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39504146"
---
# <a name="enable-password-hash-synchronization-to-azure-active-directory-domain-services"></a>啟用 Azure Active Directory Domain Services 的密碼雜湊同步
在先前工作中，您已啟用 Azure Active Directory (Azure AD) 租用戶的 Azure Active Directory Domain Services。 下一項工作是啟用 NT LAN Manager (NTLM) 和 Kerberos 驗證所需的密碼雜湊與 Azure AD Domain Services 的同步。 設定密碼雜湊同步後，使用者即可使用他們的公司認證來登入受控網域。

僅限雲端使用者帳戶，與使用 Azure AD Connect 從內部部署目錄同步的使用者帳戶，其所含的步驟不同。 

<br>
| **使用者帳戶類型** | **要執行的步驟** |
| --- |---|
| **已在 Azure AD 中建立雲端使用者帳戶** |**&#x2713;** [依照本文的指示](active-directory-ds-getting-started-password-sync.md#task-5-enable-password-hash-synchronization-to-your-managed-domain-for-cloud-only-user-accounts) |
| **已從內部部署目錄同步使用者帳戶** |**&#x2713;** [將從您內部部署 AD 同步之使用者帳戶的密碼雜湊同步到受控網域](active-directory-ds-getting-started-password-sync-synced-tenant.md) | 

<br>

> [!TIP]
> **您需要完成這兩組步驟。**
> 如果您的 Azure AD 租用戶同時具有僅限雲端使用者以及來自您內部部署 AD 的使用者，則需要完成這兩組步驟。
>

## <a name="task-5-enable-password-hash-synchronization-to-your-managed-domain-for-cloud-only-user-accounts"></a>工作 5：針對僅限雲端使用者帳戶啟用與受控網域的密碼雜湊同步
若要在受控網域上驗證使用者，Azure Active Directory Domain Services 需要適合於 NTLM 和 Kerberos 驗證之格式的密碼雜湊。 除非您針對租用戶啟用 Azure Active Directory Domain Services，否則 Azure AD 不會以 NTLM 或 Kerberos 驗證所需的格式產生或儲存密碼雜湊。 基於明顯的安全性考量，Azure AD 也不會儲存任何純文字形式的密碼認證。 因此，Azure AD 無法根據使用者的現有認證自動產生這些 NTLM 或 Kerberos 密碼雜湊。

> [!NOTE]
> **如果您的組織有僅限雲端使用者帳戶，則需要使用 Azure Active Directory Domain Services 的所有使用者必須變更其密碼。** 僅限雲端使用者帳戶是您使用 Azure 入口網站或 Azure AD PowerShell Cmdlet 在 Azure AD 目錄中建立的帳戶。 這類使用者帳戶不是從內部部署目錄進行同步。
>
>

此密碼變更程序會在 Azure AD 中產生 Azure Active Directory Domain Services 進行 Kerberos 和 NTLM 驗證所需的密碼雜湊。 您可以針對租用戶中所有需要使用 Azure Active Directory Domain Services 的使用者來使其密碼過期，或者指示他們變更其密碼。

### <a name="enable-ntlm-and-kerberos-password-hash-generation-for-a-cloud-only-user-account"></a>針對僅限雲端使用者帳戶啟用 NTLM 和 Kerberos 密碼雜湊產生功能
以下是您需要提供使用者才能變更其密碼的指示：

1. 移至貴組織的 [Azure AD 存取面板](http://myapps.microsoft.com) 頁面。

    ![啟動 Azure AD 存取面板](./media/active-directory-domain-services-getting-started/access-panel.png)

2. 在右上角，按一下您的名稱，然後從功能表中選取 [設定檔]。

    ![選取設定檔](./media/active-directory-domain-services-getting-started/select-profile.png)

3. 在 [設定檔] 頁面上，按一下 [變更密碼]。

    ![按一下 [變更密碼]](./media/active-directory-domain-services-getting-started/user-change-password.png)

   > [!TIP]
   > 如果在 [存取面板] 視窗中未顯示 [變更密碼] 選項，請確定您的組織已設定 [Azure AD 中的密碼管理](../active-directory/authentication/quickstart-sspr.md)。
   >
   >
4. 在 [變更密碼] 頁面上，輸入您現有 (舊的) 密碼，然後輸入新密碼並加以確認。

    ![使用者變更密碼](./media/active-directory-domain-services-getting-started/user-change-password2.png)

5. 按一下 [提交] 。

當您變更密碼幾分鐘之後，新的密碼就能在 Azure Active Directory Domain Services 中使用。 大約 20 分鐘之後，您就能使用最近變更的密碼來登入已加入受控網域的電腦。

## <a name="related-content"></a>相關內容
* [如何更新自己的密碼](../active-directory/user-help/active-directory-passwords-update-your-own-password.md)
* [在 Azure AD 中開始使用密碼管理](../active-directory/authentication/quickstart-sspr.md)
* [為同步處理的 Azure AD 租用戶啟用 Azure Active Directory Domain Services 的密碼雜湊同步](active-directory-ds-getting-started-password-sync-synced-tenant.md)
* [受 Azure Active Directory Domain Services 受控的網域](active-directory-ds-admin-guide-administer-domain.md)
* [將 Windows 虛擬機器加入 Azure Active Directory Domain Services 受控網域](active-directory-ds-admin-guide-join-windows-vm.md)
* [將 Red Hat Enterprise Linux 虛擬機器加入 Azure Active Directory Domain Services 受控網域](active-directory-ds-admin-guide-join-rhel-linux-vm.md)
