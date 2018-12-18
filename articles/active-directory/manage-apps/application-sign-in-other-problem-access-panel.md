---
title: 從存取面板登入應用程式的問題 | Microsoft Docs
description: 如何為從 Microsoft Azure AD 存取面板 (myapps.microsoft.com) 存取應用程式時發生的問題疑難排解
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
ms.date: 07/11/2017
ms.author: barbkess
ms.reviewer: japere
ms.openlocfilehash: 830fc1d96825e28aad41aac9afee499b9bc1f7ae
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44355189"
---
# <a name="problems-signing-in-to-an-application-from-the-access-panel"></a>從存取面板登入應用程式的問題

存取面板是網頁型入口網站，可讓在 Azure Active Directory (Azure AD) 中具有公司或學校帳戶的使用者，檢視和啟動 Azure AD 系統管理員已授權他們存取的雲端式應用程式。 

在 Azure AD 入口網站中可代表使用者設定這些應用程式。 應用程式必須經過正確的設定並指派至使用者或使用者所屬的群組，才能在存取面板中顯示該應用程式。

使用者能看見的應用程式類型可分為以下類別：

-   Office 365 應用程式

-   已設定同盟 SSO 的 Microsoft 及第三方應用程式

-   密碼型 SSO 應用程式

-   含現有 SSO 解決方案的應用程式

## <a name="general-issues-to-check-first"></a>首先檢查的一般問題

-   請確定您使用的**瀏覽器**符合存取面板的最低需求。

-   確定使用者的瀏覽器已將應用程式的 URL 新增至其**信任的網站**。

-   檢查應用程式以確認其**設定**正確。

-   確定使用者的帳戶**已啟用**可供登入。

-   確定使用者的帳戶**未鎖定**。

-   確定使用者的**密碼未過期或忘記**。

-   確定 **Multi-Factor Authentication** 未封鎖使用者存取。

-   確定**條件式存取原則**或**身分識別保護**原則未封鎖使用者存取。

-   確定使用者的**驗證連絡資訊**為最新版本，而可強制執行 Multi-Factor Authentication 或條件式存取原則。

-   也要確定嘗試清除瀏覽器的 Cookie，然後嘗試再次登入。

## <a name="meeting-browser-requirements-for-the-access-panel"></a>符合存取面板的瀏覽器需求

存取面板需要支援 JavaScript 且已啟用 CSS 的瀏覽器。 若要在存取面板中使用密碼單一登入 (SSO)，必須在使用者的瀏覽器中安裝存取面板延伸模組。 當使用者選取已設定密碼 SSO 的應用程式時，就會自動下載此存取面板延伸模組。

若是密碼 SSO，則使用者的瀏覽器可以是：

-   Internet Explorer 8、9、10、11 -- 在 Windows 7 或更新版本上

-   Windows 10 Anniversary Edition 或更新版本上的 Edge

-   Chrome - 在 Windows 7 或更新版本，和在 MacOS X 或更新版本上

-   Firefox 26.0 或更新版本 - 在 Windows XP SP2 或更新版本，和在 Mac OS X 10.6 或更新版本上

## <a name="how-to-install-the-access-panel-browser-extension"></a>如何安裝存取面板的瀏覽器延伸模組

若要安裝存取面板的瀏覽器延伸模組，請遵循下列步驟：

1.  在其中一種支援的瀏覽器中開啟[存取面板](https://myapps.microsoft.com)，然後在您的 Azure AD 中以**使用者**身分登入。

2.  按一下存取面板中的 [密碼-SSO 應用程式]。

3.  在要求安裝軟體的提示中，選取 [立即安裝]。

4.  系統會根據您的瀏覽器將您導向下載連結。 將延伸模組**新增**到瀏覽器中。

5.  如果您的瀏覽器要求，請選取 [啟用] 或 [允許] 該延伸模組。

6.  安裝之後，**重新啟動**瀏覽器工作階段。

7.  登入存取面板，並查看您是否可以**啟動**密碼-SSO 應用程式

您可能也會從下列直接連結中下載適用於 Chrome 和 Edge 的擴充功能：

-   [Chrome 存取面板延伸模組](https://chrome.google.com/webstore/detail/access-panel-extension/ggjhpefgjjfobnfoldnjipclpcfbgbhl)

-   [Edge 存取面板延伸模組](https://www.microsoft.com/store/apps/9pc9sckkzk84)

## <a name="how-to-configure-federated-single-sign-on-for-an-azure-ad-gallery-application"></a>如何為 Azure AD 資源庫應用程式設定同盟單一登入

Azure AD 資源庫中所有透過企業單一登入功能啟用的應用程式都提供逐步教學課程。 您可以存取[如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](https://azure.microsoft.com/documentation/articles/active-directory-saas-tutorial-list/)，以取得詳細的逐步指引。

若要設定 Azure AD 資源庫中的應用程式，您必須：

-   [從 Azure AD 資源庫新增應用程式](#add-an-application)

-   [在 Azure AD 中設定應用程式的中繼資料值 (登入 URL、識別碼、回覆 URL)](#configure-single-sign-on-for-an-application-from-the-azure-ad-gallery)

-   [選取使用者識別碼並新增要傳送到應用程式的使用者屬性](#select-user-identifier-and-add-user-attributes-to-be-sent-to-the-application)

-   [擷取 Azure AD 中繼資料與憑證](#download-the-azure-ad-metadata-or-certificate)

-   [在應用程式中設定 Azure AD 中繼資料值 (登入 URL、簽發者、登出 URL 與憑證)](#configure-single-sign-on-for-an-application-from-the-azure-ad-gallery)

-   [將使用者指派給應用程式](#assign-users-to-the-application)

### <a name="add-an-application-from-the-azure-ad-gallery"></a>從 Azure AD 資源庫新增應用程式

若要從 Azure AD 資源庫新增應用程式，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com)，然後以**全域管理員**或**共同管理員**身分登入

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [企業應用程式] 窗格右上角的 [新增] 按鈕。

6.  在 [從資源庫新增] 區段的 [輸入名稱] 文字方塊中，輸入應用程式名稱。

7.  選取您要設為單一登入的應用程式。

8.  新增應用程式之前，您可以從 [名稱] 文字方塊變更其名稱。

9.  按一下 [新增] 按鈕新增應用程式。

稍候片刻，您便能看見應用程式的設定窗格。

### <a name="configure-single-sign-on-for-an-application-from-the-azure-ad-gallery"></a>從 Azure AD 資源庫為應用程式設定單一登入

若要設定應用程式使用單一登入，請依照下列步驟執行：

1.  <span id="_Hlk477187909" class="anchor"><span id="_Hlk477001983" class="anchor"></span></span>開啟 [**Azure 入口網站**](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

  * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您要設定單一登入的應用程式。

7.  應用程式載入後，按一下應用程式的左側導覽功能表中的 [單一登入]。

8.  從 [模式] 下拉式清單選取 [SAML 登入]。

9.  在 [網域及 URL] 中輸入必要值。 這些值應從應用程式廠商處取得。

   1. 若要將應用程式設定為 SP 啟始的 SSO，則登入 URL 為必要值。 對於某些應用程式，識別碼也是必要值。

   2. 若要將應用程式設定為 IdP 啟始的單一登入，則回應 URL 為必要值。 對於某些應用程式，識別碼也是必要值。

10. **選擇性：** 如果您想要看到非必要值，請按一下 [顯示進階 URL 設定]。

11. 在 [使用者屬性] 中，從 [使用者識別碼] 下拉式清單選取使用者的唯一識別碼。

12. **選擇性：** 按一下 [檢視和編輯所有其他使用者屬性]，以編輯當使用者登入時要以 SAML 權杖傳送至應用程式的屬性。

   若要新增屬性︰

   1. 按一下 [新增屬性]。 輸入 [名稱]，然後從下拉式清單中選取 [值]。

   2. 按一下 [儲存]。 您會在資料表中看到新屬性。

13. 按一下 [設定 &lt;應用程式名稱&gt;]，以存取如何在應用程式中設定單一登入的文件。 此外，您會有透過該應用程式設定 SSO 所需的中繼資料 URL 與憑證。

14. 按一下 **[儲存]** 儲存組態。

15. 將使用者指派給應用程式。

### <a name="select-user-identifier-and-add-user-attributes-to-be-sent-to-the-application"></a>選取使用者識別碼並新增要傳送到應用程式的使用者屬性

若要選取使用者識別碼或新增使用者屬性，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

  * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您已設定單一登入的應用程式。

7.  應用程式載入後，按一下應用程式的左側導覽功能表中的 [單一登入]。

8.  在 [使用者屬性] 區段下，從 [使用者識別碼] 下拉式清單選取使用者的唯一識別碼。 所選的選項必須符合應用程式中預期的值，才能驗證使用者。

    >[!NOTE]
    >Azure AD 會根據應用程式在 SAML AuthRequest 中選取的值或要求的格式，來選取 NameID 屬性 (使用者識別碼) 的格式。 如需詳細資訊，請參閱[單一登入 SAML 通訊協定](https://docs.microsoft.com/azure/active-directory/develop/active-directory-single-sign-on-protocol-reference#authnrequest)文章中的＜NameIDPolicy＞一節。
    >
    >

9.  若要新增使用者屬性，按一下 [檢視和編輯所有其他使用者屬性]，以編輯當使用者登入時要以 SAML 權杖傳送至應用程式的屬性。

   若要新增屬性︰

   1. 按一下 [新增屬性]。 輸入 [名稱]，然後從下拉式清單中選取 [值]。

   2. 按一下 [儲存]。 您會在資料表中看到新屬性。

### <a name="download-the-azure-ad-metadata-or-certificate"></a>下載 Azure AD 中繼資料或憑證

若要從 Azure AD 下載應用程式中繼資料或憑證，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

  * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您已設定單一登入的應用程式。

7.  應用程式載入後，按一下應用程式的左側導覽功能表中的 [單一登入]。

8.  移至 [SAML 簽署憑證] 區段，然後按一下 [下載] 資料行值。 根據應用程式設定單一登入時所需的項目，您會看到下載中繼資料 XML 或憑證的選項。

    Azure AD 不提供取得中繼資料的 URL。 中繼資料只能擷取為 XML 檔案。

## <a name="how-to-configure-federated-single-sign-on-for-a-non-gallery-application"></a>如何為不在資源庫內的應用程式設定同盟單一登入

若要設定不在資源庫內的應用程式，您必須有 Azure AD Premium，且應用程式必須支援 SAML 2.0。 如需有關 Azure AD 版本的詳細資訊，請參閱 [Azure AD 定價](https://azure.microsoft.com/pricing/details/active-directory/)。

-   [在 Azure AD 中設定應用程式的中繼資料值 (登入 URL、識別碼、回覆 URL)](#configuring-single-sign-on)

-   [選取使用者識別碼並新增要傳送到應用程式的使用者屬性](#select-user-identifier-and-add-user-attributes-to-be-sent-to-the-application)

-   [擷取 Azure AD 中繼資料與憑證](#download-the-azure-ad-metadata-or-certificate)

-   [在應用程式中設定 Azure AD 中繼資料值 (登入 URL、簽發者、登出 URL 與憑證)](#configuring-single-sign-on)

### <a name="configure-the-applications-metadata-values-in-azure-ad-sign-on-url-identifier-reply-url"></a>在 Azure AD 中設定應用程式的中繼資料值 (登入 URL、識別碼、回覆 URL)

若要為不在 Azure AD 資源庫中的應用程式設定單一登入，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [企業應用程式] 窗格右上角的 [新增] 按鈕。

6.  按一下 [新增您自己的應用程式] 區段中的 [不在資源庫內的應用程式]。

7.  在 [名稱] 文字方塊中輸入應用程式的名稱。

8.  按一下 [新增] 按鈕新增應用程式。

9.  應用程式載入後，按一下應用程式的左側導覽功能表中的 [單一登入]。

10. 在 [模式] 下拉式清單中選取 [SAML 登入]

11. 在 [網域及 URL] 中輸入必要值 這些值應從應用程式廠商處取得。

  1. 若要將應用程式設定為 IdP 啟始的 SSO，請輸入回覆 URL 與識別碼。

  2. **選擇性：** 若要將應用程式設定為 SP 啟始的 SSO，則登入 URL 為必要值。

12. 在 [使用者屬性] 中，從 [使用者識別碼] 下拉式清單選取使用者的唯一識別碼。

13. **選擇性：** 按一下 [檢視和編輯所有其他使用者屬性]，以編輯當使用者登入時要以 SAML 權杖傳送至應用程式的屬性。

   若要新增屬性︰

   1. 按一下 [新增屬性]。 輸入 [名稱]，然後從下拉式清單中選取 [值]。

   2. 按一下 [儲存]。 您會在資料表中看到新屬性。

14. 按一下 [設定 &lt;應用程式名稱&gt;]，以存取如何在應用程式中設定單一登入的文件。 此外，您有應用程式所需的 Azure AD URL 與憑證。

### <a name="select-user-identifier-and-add-user-attributes-to-be-sent-to-the-application"></a>選取使用者識別碼並新增要傳送到應用程式的使用者屬性

若要選取使用者識別碼或新增使用者屬性，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

  * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您已設定單一登入的應用程式。

7.  應用程式載入後，按一下應用程式的左側導覽功能表中的 [單一登入]。

8.  在 [使用者屬性] 區段下，從 [使用者識別碼] 下拉式清單選取使用者的唯一識別碼。 所選的選項必須符合應用程式中預期的值，才能驗證使用者。

   >[!NOTE]
   >Azure AD 會根據應用程式在 SAML AuthRequest 中選取的值或要求的格式，來選取 NameID 屬性 (使用者識別碼) 的格式。 如需詳細資訊，請參閱[單一登入 SAML 通訊協定](https://docs.microsoft.com/azure/active-directory/develop/active-directory-single-sign-on-protocol-reference#authnrequest)文章中的＜NameIDPolicy＞一節。
   >
   >

9.  若要新增使用者屬性，按一下 [檢視和編輯所有其他使用者屬性]，以編輯當使用者登入時要以 SAML 權杖傳送至應用程式的屬性。

   若要新增屬性︰

   1. 按一下 [新增屬性]。 輸入 [名稱]，然後從下拉式清單中選取 [值]。

   2. 按一下 [儲存] 。 您會在資料表中看到新屬性。

### <a name="download-the-azure-ad-metadata-or-certificate"></a>下載 Azure AD 中繼資料或憑證

若要從 Azure AD 下載應用程式中繼資料或憑證，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

   * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您已設定單一登入的應用程式。

7.  應用程式載入後，按一下應用程式的左側導覽功能表中的 [單一登入]。

8.  移至 [SAML 簽署憑證] 區段，然後按一下 [下載] 資料行值。 根據應用程式設定單一登入時所需的項目，您會看到下載中繼資料 XML 或憑證的選項。

    Azure AD 不提供取得中繼資料的 URL。 中繼資料只能擷取為 XML 檔案。

## <a name="how-to-configure-password-single-sign-on-for-an-azure-ad-gallery-application"></a>如何為 Azure AD 資源庫應用程式設定密碼單一登入

若要設定 Azure AD 資源庫中的應用程式，您必須：

-   [從 Azure AD 資源庫新增應用程式](#add-an-application)

-   [設定應用程式使用密碼單一登入](#configure-the-application)

### <a name="add-an-application-from-the-azure-ad-gallery"></a>從 Azure AD 資源庫新增應用程式

若要從 Azure AD 資源庫新增應用程式，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com)，然後以**全域管理員**或**共同管理員**身分登入

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [企業應用程式] 窗格右上角的 [新增] 按鈕。

6.  在 [從資源庫新增] 區段的 [輸入名稱] 文字方塊中，輸入應用程式名稱

7.  選取您要設為單一登入的應用程式

8.  新增應用程式之前，您可以從 [名稱] 文字方塊變更其名稱。

9.  按一下 [新增] 按鈕新增應用程式。

稍候片刻，您便能看見應用程式的設定窗格。

### <a name="configure-the-application-for-password-single-sign-on"></a>設定應用程式使用密碼單一登入

若要設定應用程式使用單一登入，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

 * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您要設定單一登入的應用程式

7.  應用程式載入後，按一下應用程式的左側導覽功能表中的 [單一登入]。

8.  選取 [以密碼為基礎的登入] 模式。

9.  將使用者指派至應用程式。

10. 此外，您也可以選取使用者資料列，按一下 [更新認證]，然後代表使用者輸入使用者名稱和密碼，以代表使用者提供認證。 否則，系統會提示使用者在啟動時自行輸入認證。

## <a name="how-to-configure-password-single-sign-on-for-a-non-gallery-application"></a>如何為不在資源庫內的應用程式設定密碼單一登入

若要設定 Azure AD 資源庫中的應用程式，您必須：

-   [新增不在資源庫內的應用程式](#add-a-non-gallery-application)

-   [設定應用程式使用密碼單一登入](#configure-the-application-for-password-single-sign-on)

### <a name="add-a-non-gallery-application"></a>新增不在資源庫內的應用程式

若要從 Azure AD 資源庫新增應用程式，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com)，然後以**全域管理員**或**共同管理員**身分登入

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [企業應用程式] 窗格右上角的 [新增] 按鈕。

6.  按一下 [不在資源庫內的應用程式]。

7.  在 [名稱] 文字方塊中輸入應用程式的名稱。 選取 [新增]。

稍候片刻，您便能看見應用程式的設定窗格。

### <a name="configure-the-application-for-password-single-sign-on"></a>設定應用程式使用密碼單一登入

若要設定應用程式使用單一登入，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

 * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您要設定單一登入的應用程式。

7.  應用程式載入後，按一下應用程式的左側導覽功能表中的 [單一登入]。

8.  選取 [以密碼為基礎的登入] 模式。

9.  輸入**登入 URL**。 這是使用者輸入使用者名稱和密碼來登入的 URL。 請確保在此 URL 看得到登入欄位。

10. 將使用者指派至應用程式。

11. 此外，您也可以選取使用者資料列，按一下 [更新認證]，然後代表使用者輸入使用者名稱和密碼，以代表使用者提供認證。 否則，系統會提示使用者在啟動時自行輸入認證。

## <a name="how-to-assign-a-user-to-an-application-directly"></a>如何將使用者直接指派至應用程式

若要直接將一或多個使用者指派至應用程式，請依照下列步驟執行︰

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

  * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  從清單中選取您想要指派使用者的應用程式。

7.  應用程式載入之後，按一下應用程式左側導覽功能表中的 [使用者和群組]。

8.  按一下 [使用者和群組] 清單頂端的 [新增] 按鈕，以開啟 [新增指派] 窗格。

9.  按一下 [新增指派] 窗格中的 [使用者和群組] 選取器。

10. 在 [依姓名或電子郵件地址搜尋] 搜尋方塊中，輸入您有興趣指派之使用者的**全名**或**電子郵件地址**。

11. 將滑鼠停留在清單中的**使用者**上方，以顯示**核取方塊**。 按一下使用者設定檔照片或標誌旁邊的核取方塊，將使用者新增至 [已選取] 清單。

12. **選擇性︰** 如果您想要**新增多位使用者**，請在 [依姓名或電子郵件地址搜尋] 搜尋方塊中，輸入另一個**全名**或**電子郵件地址**，然後按一下核取方塊，將此使用者新增至 [已選取] 清單。

13. 當您完成選取使用者時，按一下 [選取] 按鈕，將他們新增到要指派至應用程式的使用者和群組清單。

14. **選擇性︰** 按一下 [新增指派] 窗格中的 [選取角色] 選取器，以選取要指派給您已選取之使用者的角色。

15. 按一下 [指派] 按鈕，將應用程式指派給選取的使用者。

稍待片刻，您已選取的使用者便能在存取面板中啟動這些應用程式。

## <a name="if-these-troubleshooting-steps-do-not-the-resolve-the-issue"></a>如果這些疑難排解步驟無法解決問題

使用下列資訊 (若有的話) 開啟支援票證︰

-   相互關聯錯誤 ID

-   UPN (使用者電子郵件地址)

-   TenantID

-   瀏覽器類型

-   錯誤發生期間的時區和時間/時間範圍

-   Fiddler 追蹤

## <a name="next-steps"></a>後續步驟
[使用應用程式 Proxy 提供單一登入應用程式](application-proxy-configure-single-sign-on-with-kcd.md)

