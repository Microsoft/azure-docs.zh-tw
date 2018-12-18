---
title: 為 Azure AD Domain Services 受控網域建立安全 LDAP 憑證 | Microsoft Docs
description: 為 Azure AD Domain Services 受控網域建立安全 LDAP 憑證
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
ms.date: 08/01/2017
ms.author: maheshu
ms.openlocfilehash: d34da3c7a32a9fd425e30880ba9bc808d9d2bab1
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39505979"
---
# <a name="create-a-pfx-file-with-the-secure-ldap-ldaps-certificate-for-a-managed-domain"></a>為受控網域建立具有安全 LDAP (LDAPS) 憑證的 .PFX 檔案

## <a name="before-you-begin"></a>開始之前
完成[工作 1：取得安全 LDAP 的憑證](active-directory-ds-admin-guide-configure-secure-ldap.md)。


## <a name="task-2-export-the-secure-ldap-certificate-to-a-pfx-file"></a>工作 2：將安全 LDAP 憑證匯出到 .PFX 檔案
開始此工作之前，請先從公開憑證授權單位取得安全 LDAP 憑證，或建立自我簽署憑證。

將 LDAPS 憑證匯出到.PFX 檔案：

1. 按 [開始] 按鈕並輸入 **R**。在 [執行] 對話方塊中，輸入 **mmc**，然後按一下 [確定]。

    ![啟動 MMC 主控台](./media/active-directory-domain-services-admin-guide/secure-ldap-start-run.png)
2. 在 [使用者帳戶控制] 提示字元處按一下 [是]，以系統管理員身分啟動 MMC (Microsoft Management Console)。
3. 在 [檔案] 功能表上，按一下 [新增/移除嵌入式管理單元...]。

    ![新增嵌入式管理單元至 MMC 主控台](./media/active-directory-domain-services-admin-guide/secure-ldap-add-snapin.png)
4. 在 [新增或移除嵌入式管理單元] 對話方塊中，選取 [憑證] 嵌入式管理單元，然後按一下 [新增 >] 按鈕。

    ![新增憑證嵌入式管理單元至 MMC 主控台](./media/active-directory-domain-services-admin-guide/secure-ldap-add-certificates-snapin.png)
5. 在 [憑證嵌入式管理單元] 精靈中，選取 [電腦帳戶] 並按 [下一步]。

    ![新增電腦帳戶的憑證嵌入式管理單元](./media/active-directory-domain-services-admin-guide/secure-ldap-add-certificates-computer-account.png)
6. 在 [選取電腦] 頁面上，選取 [本機電腦: (執行這個主控台的電腦)] 並按一下 [完成]。

    ![新增憑證嵌入式管理單元 - 選取電腦](./media/active-directory-domain-services-admin-guide/secure-ldap-add-certificates-local-computer.png)
7. 在 [新增或移除嵌入式管理單元] 對話方塊中，按一下 [確定] 以在 MMC 中新增憑證嵌入式管理單元。

    ![新增憑證嵌入式管理單元至 MMC - 完成](./media/active-directory-domain-services-admin-guide/secure-ldap-add-certificates-snapin-done.png)
8. 在 MMC 視窗中，按一下以展開 [主控台根目錄] 。 您應該會看到已載入 [憑證] 嵌入式管理單元。 按一下 [憑證 (本機電腦)]  加以展開。 按一下以依序展開 [個人] 節點和 [憑證] 節點。

    ![開啟個人憑證存放區](./media/active-directory-domain-services-admin-guide/secure-ldap-open-personal-store.png)
9. 您應該會看到我們所建立的自我簽署憑證。 您可以檢查憑證的屬性以驗證指紋符合您在建立憑證時 PowerShell 視窗上所報告的指紋。
10. 選取自我簽署憑證並 **按一下滑鼠右鍵**。 在快顯功能表中，依序選取 [所有工作] 和 [匯出...]。

    ![匯出憑證](./media/active-directory-domain-services-admin-guide/secure-ldap-export-cert.png)
11. 在 [憑證匯出精靈] 中按 [下一步]。

    ![匯出憑證精靈](./media/active-directory-domain-services-admin-guide/secure-ldap-export-cert-wizard.png)
12. 在 [匯出私密金鑰] 頁面上，選取 [是，匯出私密金鑰] 並按 [下一步]。

    ![匯出憑證的私密金鑰](./media/active-directory-domain-services-admin-guide/secure-ldap-export-private-key.png)

    > [!WARNING]
    > 您必須匯出私密金鑰以及憑證。 如果您提供的 PFX 未包含憑證的私密金鑰，則為受控網域啟用安全的 LDAP 會失敗。
    >
    >

13. 在 [匯出檔案格式] 頁面上，選取 [個人資訊交換 - PKCS #12 (.PFX)] 作為匯出憑證的檔案格式。

    ![匯出憑證檔案格式](./media/active-directory-domain-services-admin-guide/secure-ldap-export-to-pfx.png)

    > [!NOTE]
    > 只有 .PFX 檔案格式受到支援。 請勿將憑證匯出為 .CER 檔案格式。
    >
    >

14. 在 [安全性] 頁面上選取 [密碼] 選項，然後輸入密碼來保護 .PFX 檔案。 請記住此密碼，因為下一個工作會用到它。 按 [下一步] 。

    ![憑證匯出的密碼 ](./media/active-directory-domain-services-admin-guide/secure-ldap-export-select-password.png)

    > [!NOTE]
    > 記下此密碼。 在[工作 3 - 為受控網域啟用安全 LDAP](active-directory-ds-admin-guide-configure-secure-ldap-enable-ldaps.md) 中針對此受控網域啟用安全 LDAP 時，需要這個密碼
    >
    >

15. 在 [要匯出的檔案]  頁面上，指定檔案名稱及接收匯出憑證的位置。

    ![憑證匯出的路徑](./media/active-directory-domain-services-admin-guide/secure-ldap-export-select-path.png)
16. 在接下來的頁面上按一下 [完成]  ，以將憑證匯出至 PFX 檔案。 憑證匯出後，您應該會看到確認對話方塊。

    ![憑證匯出完成](./media/active-directory-domain-services-admin-guide/secure-ldap-exported-as-pfx.png)


## <a name="next-step"></a>後續步驟
[工作 3：為受控網域啟用安全 LDAP](active-directory-ds-admin-guide-configure-secure-ldap-enable-ldaps.md)
