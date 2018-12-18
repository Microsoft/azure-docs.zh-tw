---
title: 登入 Microsoft 應用程式的問題 | Microsoft Docs
description: 為使用 Azure AD 登入第一方 Microsoft 應用程式 (如 Office 365) 時遇到的問題疑難排解
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
ms.assetid: ''
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/10/2018
ms.author: barbkess
ms.reviewer: asteen
ms.openlocfilehash: ab0face3c211a8ea1477f40167f80330f0423498
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44379259"
---
# <a name="problems-signing-in-to-a-microsoft-application"></a>登入 Microsoft 應用程式的問題

Microsoft 應用程式 (如 Office 365 Exchange、SharePoint、Yammer 等) 在指派與管理方面，比第三方 SaaS 應用程式或與 Azure AD 整合以單一登入的其他應用程式略為困難。

使用者有三種方式可取得 Microsoft 所發佈應用程式的存取權。

-   對於 Office 365 或其他付費套件中的應用程式，會透過**指派授權**直接指派至使用者帳戶，或使用我們的群組授權指派功能來透過群組，將存取權授與使用者。

-   對於 Microsoft 或第三方免費發佈給任何人使用的應用程式，會透過**使用者同意**將存取權授與使用者。 這表示使用者透過其 Azure AD 工作或學校帳戶登入應用程式，並允許應用程式能夠存取其帳戶的某些受限制資料集。

-   對於 Microsoft 或第三方免費發佈給任何人使用的應用程式，也可透過**系統管理員同意**將存取權授與使用者。 這表示系統管理員已決定組織中的每個人都能使用該應用程式，因此系統管理員使用全域管理員帳戶身分登入應用程式，並將存取權授與組織中的每個人。

若要為您的問題疑難排解，請先從[一般應用程式存取問題考量事項](#general-problem-areas-with-application-access-to-consider)開始，然後參閱[逐步解說：Microsoft 應用程式存取疑難排解步驟](#walkthrough-steps-to-troubleshoot-microsoft-application-access)以了解詳細資料。

## <a name="general-problem-areas-with-application-access-to-consider"></a>一般應用程式存取問題考量事項

以下為一般問題清單，如果您知道從何處開始疑難排解，您可以從這個清單向下切入，但建議您參閱以下的逐步解說以快速進行：[逐步解說：Microsoft 應用程式存取疑難排解步驟](#walkthrough-steps-to-troubleshoot-microsoft-application-access)。

-   [使用者帳戶的問題](#problems-with-the-users-account)

-   [群組的問題](#problems-with-groups)

-   [條件式存取原則的問題](#problems-with-conditional-access-policies)

-   [應用程式同意的問題](#problems-with-application-consent)

## <a name="steps-to-troubleshoot-microsoft-application-access"></a>Microsoft 應用程式存取疑難排解步驟

以下是一些使用者無法登入 Microsoft 應用程式的常見問題。

-   首先檢查的一般問題

  * 確定使用者登入的是**正確的 URL**，不是本機應用程式 URL。

  * 確定使用者的帳戶**未鎖定**。

  * 確定 Azure Active Directory 中**有使用者的帳戶存在**。 [檢查 Azure Active Directory 中是否存在使用者帳戶](#problems-with-the-users-account)

  * 確定使用者的帳戶**已啟用**可供登入。[檢查使用者帳戶的狀態](#problems-with-the-users-account)

  * 確定使用者的**密碼未過期或忘記**。 [重設使用者密碼](#reset-a-users-password)或[啟用自助式密碼重設](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-getting-started)

   * 確定 **Multi-Factor Authentication** 未封鎖使用者存取。 [檢查使用者的多重要素驗證狀態](#check-a-users-multi-factor-authentication-status)或[檢查使用者的驗證連絡資訊](#check-a-users-authentication-contact-info)

   * 確定**條件式存取原則**或**身分識別保護**原則未封鎖使用者存取。 [檢查特定條件式存取原則](#problems-with-conditional-access-policies)、[檢查特定應用程式條件式存取原則](#check-a-specific-applications-conditional-access-policy)或[停用特定條件式存取原則](#disable-a-specific-conditional-access-policy)

   * 確定使用者的**驗證連絡資訊**為最新版本，而可強制執行 Multi-Factor Authentication 或條件式存取原則。 [檢查使用者的多重要素驗證狀態](#check-a-users-multi-factor-authentication-status)或[檢查使用者的驗證連絡資訊](#check-a-users-authentication-contact-info)

-   對於需要授權的 **Microsoft** **應用程式** (如 Office365)，以下是一些當您排除上述一般問題後要檢查的特定問題：

   * 確定使用者已獲**授權指派**。 [檢查使用者獲指派的授權](#check-a-users-assigned-licenses)或[檢查群組獲指派的授權](#check-a-groups-assigned-licenses)

   * 如果將授權**指派至****靜態群組**，請確定**使用者屬於**該群組。 [檢查使用者的群組成員資格](#check-a-users-group-memberships)

   * 如果授權**指派至****動態群組**，請確定**動態群組規則設定正確**。 [檢查動態群組成員資格準則](#check-a-dynamic-groups-membership-criteria)

   * 如果授權**指派至****動態群組**，請確定動態群組已**完成處理**其成員資格，且**使用者為成員** (這可能需要一些時間)。 [檢查使用者的群組成員資格](#check-a-users-group-memberships)

   *  一旦您確定授權已指派，請確定授權**尚未過期**。

   *  確定授權適用於目前正在存取的**應用程式**。

-   對於不需要授權的 **Microsoft** **應用程式**，以下為其他的檢查事項：

   * 如果應用程式要求的是**使用者層級權限** (例如「存取此使用者的信箱」)，請確定使用者已登入該應用程式，並已執行**使用者層級同意作業**，讓應用程式可存取其資料。

   * 如果應用程式要求的是**系統管理員層級權限** (例如「存取所有使用者的信箱」)，請確定全域管理員已**代表組織中所有使用者執行系統管理員層級同意作業**。

## <a name="problems-with-the-users-account"></a>使用者帳戶的問題

當指派至應用程式的使用者有問題時，可封鎖應用程式存取權。 以下是針對使用者和其帳戶設定進行疑難排解的一些方法︰

-   [檢查 Azure Active Directory 中是否存在使用者帳戶](#check-if-a-user-account-exists-in-azure-active-directory)

-   [檢查使用者帳戶的狀態](#check-a-users-account-status)

-   [重設使用者的密碼](#reset-a-users-password)

-   [啟用自助密碼重設](#enable-self-service-password-reset)

-   [檢查使用者的多重要素驗證狀態](#check-a-users-multi-factor-authentication-status)

-   [檢查使用者的驗證連絡資訊](#check-a-users-authentication-contact-info)

-   [檢查使用者的群組成員資格](#check-a-users-group-memberships)

-   [檢查使用者獲指派的授權](#check-a-users-assigned-licenses)

-   [指派授權至使用者](#assign-a-user-a-license)

### <a name="check-if-a-user-account-exists-in-azure-active-directory"></a>檢查 Azure Active Directory 中是否存在使用者帳戶

若要檢查使用者的帳戶是否存在，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [所有使用者]。

6.  **搜尋**您感興趣的使用者，**按一下資料列**選取該使用者。

7.  檢查使用者物件的屬性，確定符合您的預期，沒有遺失資料。

### <a name="check-a-users-account-status"></a>檢查使用者帳戶的狀態

若要檢查使用者的帳戶狀態，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [所有使用者]。

6.  **搜尋**您感興趣的使用者，**按一下資料列**選取該使用者。

7.  按一下 [設定檔]。

8.  在 [設定] 下，確定 [封鎖登入] 設為 [否]。

### <a name="reset-a-users-password"></a>重設使用者的密碼

若要重設使用者的密碼，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [所有使用者]。

6.  **搜尋**您感興趣的使用者，**按一下資料列**選取該使用者。

7.  按一下使用者窗格頂端的 [重設密碼] 按鈕。

8.  在出現的 [重設密碼] 窗格中，按一下 [重設密碼] 按鈕。

9.  為使用者複製**暫時密碼**或**輸入新密碼**。

10. 將這個新的密碼告知使用者，他們下次登入 Azure Active Directory 時必須變更此密碼。

### <a name="enable-self-service-password-reset"></a>啟用自助式密碼重設

若要啟用自助式密碼重設，請遵循下列部署步驟︰

-   [讓使用者重設其 Azure Active Directory 密碼](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-getting-started#enable-users-to-reset-their-azure-ad-passwords)

-   [讓使用者重設或變更其 Active Directory 內部部署密碼](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-getting-started#enable-users-to-reset-or-change-their-ad-passwords)

### <a name="check-a-users-multi-factor-authentication-status"></a>檢查使用者的多重要素驗證狀態

若要檢查使用者的多重要素驗證狀態，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [所有使用者]。

6.  按一下窗格頂端的 [Multi-Factor Authentication] 按鈕。

7.  當 **Multi-Factor Authentication 管理網站**載入後，請確定您位於 [使用者] 索引標籤上。

8.  在使用者清單中搜尋、篩選或排序來尋找使用者。

9.  從使用者清單中選取使用者，然後視需要 [啟用]、[停用] 或 [強制執行] 多重要素驗證。

  * **注意**：如果使用者處理 [已強制] 狀態，您可以暫時將他們設為 [已停用]，讓他們回到各自的帳戶。 退回之後，您可以再次將其狀態變更為 [已啟用]，以要求他們在下次登入時重新註冊連絡資訊。 或者，您可以依照[檢查使用者的驗證連絡資訊](#check-a-users-authentication-contact-info)中的步驟，為他們確認或設定此資料。

### <a name="check-a-users-authentication-contact-info"></a>檢查使用者的驗證連絡資訊

若要檢查用於多重要素驗證、條件式存取、身分識別保護和密碼重設的使用者驗證連絡資訊，請遵循下列步驟︰

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [所有使用者]。

6.  **搜尋**您感興趣的使用者，**按一下資料列**選取該使用者。

7.  按一下 [設定檔]。

8.  向下捲動至 [驗證連絡資訊]。

9.  **檢閱**使用者註冊的資料，並視需要予以更新。

### <a name="check-a-users-group-memberships"></a>檢查使用者的群組成員資格

若要檢查使用者的群組成員資格，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [所有使用者]。

6.  **搜尋**您感興趣的使用者，**按一下資料列**選取該使用者。

7.  按一下 [群組]，查看使用者是哪些群組的成員。

### <a name="check-a-users-assigned-licenses"></a>檢查使用者獲指派的授權

若要檢查使用者獲指派的授權，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [所有使用者]。

6.  **搜尋**您感興趣的使用者，**按一下資料列**選取該使用者。

7.  按一下 [授權]，查看使用者目前已指派的授權。

### <a name="assign-a-user-a-license"></a>指派授權至使用者 

若要指派授權至使用者，請依遵循下列步驟︰

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [所有使用者]。

6.  **搜尋**您感興趣的使用者，**按一下資料列**選取該使用者。

7.  按一下 [授權]，查看使用者目前已指派的授權。

8.  按一下 [指派] 按鈕。

9.  從可用產品清單中選取**一或多個產品**。

10. (**選擇性**) 按一下 [指派選項] 項目，更細微地指派產品。 完成時按一下 [確定]。

11. 按一下 [指派] 按鈕，將這些授權指派至這位使用者。

## <a name="problems-with-groups"></a>群組的問題

當指派至應用程式的群組有問題時，可封鎖應用程式存取權。 以下是針對群組和群組成員資格問題疑難排解的一些方法︰

-   [檢查群組成員資格](#check-a-groups-membership)

-   [檢查動態群組成員資格準則](#check-a-dynamic-groups-membership-criteria)

-   [檢查群組獲指派的授權](#check-a-groups-assigned-licenses)

-   [重新處理群組的授權](#reprocess-a-groups-licenses)

-   [指派授權至群組](#assign-a-group-a-license)

### <a name="check-a-groups-membership"></a>檢查群組成員資格

若要檢查群組的成員資格，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [所有群組]。

6.  **搜尋**您感興趣的群組，然後**按一下資料列**加以選取。

7.  按一下 [成員] 以檢閱已指派至此群組的使用者清單。

### <a name="check-a-dynamic-groups-membership-criteria"></a>檢查動態群組成員資格準則 

若要檢查動態群組的成員資格準則，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [所有群組]。

6.  **搜尋**您感興趣的群組，然後**按一下資料列**加以選取。

7.  按一下 [動態成員資格規則]。

8.  檢閱為此群組定義的**簡單**或 **進階**規則，確定您要成為此群組成員的使用者符合這些準則。

### <a name="check-a-groups-assigned-licenses"></a>檢查群組獲指派的授權

若要檢查群組獲指派的授權，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [所有群組]。

6.  **搜尋**您感興趣的群組，然後**按一下資料列**加以選取。

7.  按一下 [授權]，以查看目前已指派至群組的授權。

### <a name="reprocess-a-groups-licenses"></a>重新處理群組的授權

若要重新處理群組獲指派的授權，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [所有群組]。

6.  **搜尋**您感興趣的群組，然後**按一下資料列**加以選取。

7.  按一下 [授權]，以查看目前已指派至群組的授權。

8.  按一下 [重新處理] 按鈕，以確定指派至此群組成員的授權是最新的。 根據群組的大小和複雜度，這可能需要很長的時間。

   >[!NOTE]
   >若要更快速執行此作業，請考慮暫時將授權直接指派至使用者。 [指派授權至使用者](#problems-with-application-consent)。
   >
   >

### <a name="assign-a-group-a-license"></a>指派授權至群組

若要將授權指派給群組，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [所有群組]。

6.  **搜尋**您感興趣的群組，然後**按一下資料列**加以選取。

7.  按一下 [授權]，以查看目前已指派至群組的授權。

8.  按一下 [指派] 按鈕。

9.  從可用產品清單中選取**一或多個產品**。

10. (**選擇性**) 按一下 [指派選項] 項目，更細微地指派產品。 完成時按一下 [確定]。

11. 按一下 [指派] 按鈕，將這些授權指派至這個群組。 根據群組的大小和複雜度，這可能需要很長的時間。

   >[!NOTE]
   >若要更快速執行此作業，請考慮暫時將授權直接指派至使用者。 [指派授權至使用者](#problems-with-application-consent)。
   > 
   >

## <a name="problems-with-conditional-access-policies"></a>條件式存取原則的問題

### <a name="check-a-specific-conditional-access-policy"></a>檢查特定的條件式存取原則

若要檢查或驗證單一條件式存取原則︰

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [企業應用程式]。

5.  按一下 [條件式存取] 瀏覽項目。

6.  按一下您想要檢查的原則。

7.  檢查有無特定的條件、指派，或其他可能會封鎖使用者存取的設定。

   >[!NOTE]
   >您可能會想要暫時停用此原則，以確保不會影響登入。若要這麼做，請設定 [啟用原則] 切換至 [否]，然後按一下 [儲存] 按鈕。
   >
   >

### <a name="check-a-specific-applications-conditional-access-policy"></a>檢查特定應用程式的條件式存取原則

若要檢查或驗證單一應用程式目前設定的條件式存取原則︰

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [企業應用程式]。

5.  按一下 [所有應用程式]。

6.  搜尋您感興趣的應用程式，或依應用程式顯示名稱或應用程式識別碼搜尋使用者嘗試登入的應用程式。

     >[!NOTE]
     >如果您看不到正在尋找的應用程式，請按一下 [篩選] 按鈕，然後將清單範圍展開至 [所有應用程式]。 如果您想要看到更多的資料行，請按一下 [資料行] 按鈕為應用程式新增其他詳細資料。
     >
     >

7.  按一下 [條件式存取] 瀏覽項目。

8.  按一下您想要檢查的原則。

9.  檢查有無特定的條件、指派，或其他可能會封鎖使用者存取的設定。

     >[!NOTE]
     >您可能會想要暫時停用此原則，以確保不會影響登入。若要這麼做，請設定 [啟用原則] 切換至 [否]，然後按一下 [儲存] 按鈕。
     >
     >

### <a name="disable-a-specific-conditional-access-policy"></a>停用特定的條件式存取原則

若要檢查或驗證單一條件式存取原則︰

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [企業應用程式]。

5.  按一下 [條件式存取] 瀏覽項目。

6.  按一下您想要檢查的原則。

7.  設定 [啟用原則] 切換至 [否] 以停用原則，然後按一下 [儲存] 按鈕。

## <a name="problems-with-application-consent"></a>應用程式同意的問題

由於未進行適當的權限同意作業，因此會阻止應用程式存取。 以下提供一些方式，可讓您疑難排解並解決應用程式同意問題：

-   [執行使用者層級同意作業](#perform-a-user-level-consent-operation)

-   [為任何應用程式執行系統管理員層級同意作業](#perform-administrator-level-consent-operation-for-any-application)

-   [為單一租用戶應用程式執行系統管理員層級同意](#perform-administrator-level-consent-for-a-single-tenant-application)

-   [為多租用戶應用程式執行系統管理員層級同意](#perform-administrator-level-consent-for-a-multi-tenant-application)

### <a name="perform-a-user-level-consent-operation"></a>執行使用者層級同意作業

-   對於任何已啟用 Open ID Connect 且要求權限的應用程式，若瀏覽至應用程式的登入畫面，會為登入的使用者執行對該應用程式的使用者層級同意。

-   如果您想要以程式設計方式執行這項作業，請參閱[要求個別使用者同意](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-scopes#requesting-individual-user-consent)。

### <a name="perform-administrator-level-consent-operation-for-any-application"></a>為任何應用程式執行系統管理員層級同意作業

-   **僅適用於使用 V1 應用程式模型開發的應用程式**：您可以將 “**?prompt=admin\_consent**” 新增至應用程式登入 URL 的結尾，來強制進行此系統管理員層級同意。

-   **適用於任何使用 V2 應用程式模型開發的應用程式**：您可以依照[使用系統管理員同意端點](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-scopes#using-the-admin-consent-endpoint)中**向目錄管理員要求權限**一節的指示，強制進行此系統管理員層級同意。

### <a name="perform-administrator-level-consent-for-a-single-tenant-application"></a>為單一租用戶應用程式執行系統管理員層級同意

-   適用於要求權限的**單一租用戶應用程式** (例如您正在開發或您在組織中擁有的應用程式)：您可以代表所有的使用者執行**系統管理員層級同意**作業，做法是以全域管理員身分登入，然後按一下 [應用程式登錄] -&gt; [所有應用程式] -&gt; [選取應用程式] -&gt; [必要權限] 窗格頂端的 [授與權限] 按鈕。

-   **適用於任何使用 V1 或 V2 應用程式模型開發的應用程式**：您可以依照[使用系統管理員同意端點](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-scopes#using-the-admin-consent-endpoint)中**向目錄管理員要求權限**一節的指示，強制進行此系統管理員層級同意。

### <a name="perform-administrator-level-consent-for-a-multi-tenant-application"></a>為多租用戶應用程式執行系統管理員層級同意

-   適用於要求權限的**多租用戶應用程式** (例如第三方或 Microsoft 開發的應用程式)：您可以執行**系統管理員層級同意**作業。 以全域管理員身分登入，然後按一下 [企業應用程式] -&gt; [所有應用程式] -&gt; [選取應用程式] -&gt; [權限] 窗格 (即將推出) 下方的 [授與權限] 按鈕。

-   您也可以依照[使用系統管理員同意端點](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-scopes#using-the-admin-consent-endpoint)中**向目錄管理員要求權限**一節的指示，強制進行系統管理員層級同意。

## <a name="next-steps"></a>後續步驟
[使用系統管理員同意端點](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-scopes#using-the-admin-consent-endpoint)

