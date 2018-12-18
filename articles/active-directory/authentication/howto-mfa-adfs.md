---
title: 使用 Azure MFA 與 AD FS 保護雲端資源 | Microsoft Docs
description: 這是說明如何在雲端開始使用 Azure MFA 和 AD FS 的 Azure Multi-Factor Authentication 頁面。
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: afb28488fc47f018b6d192eb1b65a54499ac8ff9
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43666358"
---
# <a name="securing-cloud-resources-with-azure-multi-factor-authentication-and-ad-fs"></a>使用 Azure Multi-Factor Authentication 與 AD FS 保護雲端資源

如果您的組織與 Azure Active Directory 同盟，請使用 Azure Multi-factor Authentication 或 Active Directory Federation Services (AD FS) 來保護 Azure AD 存取的資源。 使用下列程序可利用 Azure Multi-Factor Authentication 或 Active Directory Federation Services 來保護 Azure Active Directory 資源。

## <a name="secure-azure-ad-resources-using-ad-fs"></a>使用 AD FS 保護 Azure AD 資源

若要保護雲端資源，請設定宣告規則，以便在使用者成功執行雙步驟驗證時，Active Directory Federation Services 會發出 multipleauthn 宣告。 此宣告會傳遞至 Azure AD。 遵循此程序來逐步執行各個步驟︰

1. 開啟 [AD FS 管理]。
2. 在左側選取 [信賴憑證者信任]。
3. 以滑鼠右鍵按一下 [Microsoft Office 365 身分識別平台]，然後選取 [編輯宣告規則]。

   ![雲端](./media/howto-mfa-adfs/trustedip1.png)

4. 在 [發佈轉換規則] 上，按一下 [新增規則]。

   ![雲端](./media/howto-mfa-adfs/trustedip2.png)

5. 在 [新增轉換宣告規則精靈] 上，從下拉式清單選取 [通過或篩選傳入宣告]，然後按 [下一步]。

   ![雲端](./media/howto-mfa-adfs/trustedip3.png)

6. 指定規則的名稱。 
7. 選取 [驗證方法參考] 做為傳入宣告類型。
8. 選取 [傳遞所有宣告值]。
    ![新增轉換宣告規則精靈](./media/howto-mfa-adfs/configurewizard.png)
9. 按一下 [完成]。 關閉 AD FS 管理主控台。

## <a name="trusted-ips-for-federated-users"></a>同盟使用者的可信任 IP

信任的 IP 可讓系統管理員針對特定的 IP 位址，或針對從他們自己的內部網路發出要求的同盟使用者，略過雙步驟驗證。 下列各節說明當要求是來自同盟使用者的內部網路時，如何設定同盟使用者的 Azure Multi-Factor Authentication 信任的 IP，以及略過雙步驟驗證。 這是藉由設定 AD FS 使用「通過或篩選傳入宣告」範本與「位於公司網路之內」宣告類別來達成。

此範例使用 Office 365 做為信賴憑證者信任。

### <a name="configure-the-ad-fs-claims-rules"></a>設定 AD FS 宣告規則

我們要做的第一件事是設定 AD FS 宣告。 建立兩個宣告規則，一個用於「位於公司網路之內」宣告類型，另一個用於保持使用者登入。

1. 開啟 [AD FS 管理]。
2. 在左側選取 [信賴憑證者信任]。
3. 以滑鼠右鍵按一下 **[Microsoft Office 365 身分識別平台]**，然後選取 **[編輯宣告規則...]**。
   ![雲端](./media/howto-mfa-adfs/trustedip1.png)
4. 在 [發佈轉換規則] 上，按一下 **[新增規則]**。
   ![雲端](./media/howto-mfa-adfs/trustedip2.png)
5. 在 [新增轉換宣告規則精靈] 上，從下拉式清單選取 [通過或篩選傳入宣告]，然後按 [下一步]。
   ![雲端](./media/howto-mfa-adfs/trustedip3.png)
6. 在 [宣告規則名稱] 旁邊的方塊中，命名您的規則。 例如：InsideCorpNet。
7. 從 [連入宣告類型] 旁邊的下拉式清單中，選取 [位於公司網路之內]。
   ![雲端](./media/howto-mfa-adfs/trustedip4.png)
8. 按一下 [完成] 。
9. 在 [發佈轉換規則] 上，按一下 [新增規則]。
10. 在 [新增轉換宣告規則精靈] 上，從下拉式清單選取 [使用自訂規則傳送宣告]，然後按 [下一步]。
11. 在 [宣告規則名稱] 下的方塊中：輸入「保持使用者登入」。
12. 在 [自訂規則] 方塊中輸入：

        c:[Type == "http://schemas.microsoft.com/2014/03/psso"]
            => issue(claim = c);
    ![雲端](./media/howto-mfa-adfs/trustedip5.png)
13. 按一下 [完成] 。
14. 按一下 [套用]。
15. 按一下 [確定] 。
16. 關閉 [AD FS 管理]。

### <a name="configure-azure-multi-factor-authentication-trusted-ips-with-federated-users"></a>搭配同盟使用者設定 Azure Multi-Factor Authentication 信任的 IP

既然已經有宣告，我們可以開始設定信任的 IP。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [Azure Active Directory] > [條件式存取] > [具名位置]。
3. 從 [條件式存取 - 具名位置] 刀鋒視窗，選取 [設定 MFA 信任的 IP]

   ![Azure AD 條件式存取具名位置 設定 MFA 信任的 IP](./media/howto-mfa-adfs/trustedip6.png)

4. 在 [服務設定] 頁面的 [信任的 IP] 下，選取 [對於來自內部網路之同盟使用者的要求略過多重要素驗證]。  
5. 按一下 [儲存] 。

這樣就大功告成了！ 現在，當宣告來自公司內部網路之外時，Office 365 使用者同盟只需要使用 MFA。
