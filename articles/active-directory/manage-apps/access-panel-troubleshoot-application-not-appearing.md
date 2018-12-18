---
title: 指派的應用程式未出現在存取面板上 | Microsoft Docs
description: 針對應用程式為什麼未出現在存取面板上進行疑難排解
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
ms.topic: article
ms.date: 09/09/2018
ms.author: barbkess
ms.reviwer: japere
ms.openlocfilehash: 9e3b1976cf11df77b8a5570c83f055001be8bb46
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44355205"
---
# <a name="an-assigned-application-is-not-appearing-on-the-access-panel"></a>指派的應用程式未出現在存取面板上

存取面板是網頁型入口網站，可讓在 Azure Active Directory (Azure AD) 中具有公司或學校帳戶的使用者，檢視和啟動 Azure AD 系統管理員已授權他們存取的雲端式應用程式。 在 Azure AD 入口網站中可代表使用者設定這些應用程式。 應用程式必須經過正確的設定並指派至使用者或使用者所屬的群組，才能在存取面板中顯示該應用程式。

使用者能看見的應用程式類型可分為以下類別：

-   Office 365 應用程式

-   已設定同盟 SSO 的 Microsoft 及第三方應用程式

-   密碼型 SSO 應用程式

-   含現有 SSO 解決方案的應用程式

## <a name="general-issues-to-check-first"></a>首先檢查一般問題

-   如果才剛將應用程式新增至使用者，請在幾分鐘之後嘗試登入並再次登出使用者的存取面板，以查明是否已新增該應用程式。

-   如果才剛從使用者或使用者所屬群組移除授權，則根據群組的大小和複雜度而定，可能要經過一段很長的時間，變更才會生效。 登入存取面板之前，請多等一些時間。

## <a name="problems-related-to-application-configuration"></a>應用程式設定的相關問題

應用程式未出現在使用者的存取面板上可能是因為並未正確設定。 以下提供一些方式，可讓您用來針對應用程式設定的相關問題進行疑難排解：

-   [如何為 Azure AD 資源庫應用程式設定同盟單一登入](#how-to-configure-federated-single-sign-on-for-an-azure-ad-gallery-application)

-   [如何為不在資源庫內的應用程式設定同盟單一登入](#how-to-configure-federated-single-sign-on-for-a-non-gallery-application)

-   [如何為 Azure AD 資源庫應用程式設定密碼單一登入應用程式](#how-to-configure-password-single-sign-on-for-a-non-gallery-application)

-   [如何為不在資源庫內的應用程式設定密碼單一登入應用程式](#how-to-configure-password-single-sign-on-for-a-non-gallery-application)

### <a name="how-to-configure-federated-single-sign-on-for-an-azure-ad-gallery-application"></a>如何為 Azure AD 資源庫應用程式設定同盟單一登入

Azure AD 資源庫中所有透過企業單一登入功能啟用的應用程式都提供逐步教學課程。 您可以存取[如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](https://azure.microsoft.com/documentation/articles/active-directory-saas-tutorial-list/)，以取得詳細的逐步指引。

若要設定 Azure AD 資源庫中的應用程式，您必須：

-   [從 Azure AD 資源庫新增應用程式](#add-an-application-from-the-azure-ad-gallery)

-   [在 Azure AD 中設定應用程式的中繼資料值 (登入 URL、識別碼、回覆 URL)](#configure-single-sign-on-for-an-application-from-the-azure-ad-gallery)

-   [選取使用者識別碼並新增要傳送到應用程式的使用者屬性](#select-user-identifier-and-add-user-attributes-to-be-sent-to-the-application)

-   [擷取 Azure AD 中繼資料與憑證](#download-the-azure-ad-metadata-or-certificate)

-   [在應用程式中設定 Azure AD 中繼資料值 (登入 URL、簽發者、登出 URL 與憑證)](#configure-single-sign-on-for-an-application-from-the-azure-ad-gallery)

#### <a name="add-an-application-from-the-azure-ad-gallery"></a>從 Azure AD 資源庫新增應用程式

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

#### <a name="configure-single-sign-on-for-an-application-from-the-azure-ad-gallery"></a>從 Azure AD 資源庫為應用程式設定單一登入

若要設定應用程式使用單一登入，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

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

   2. 若要將應用程式設定為 IdP 啟始的單一登入，則 [登入 URL] 為必要值。 對於某些應用程式，識別碼也是必要值。

10. **選擇性：** 如果您想要看到非必要值，請按一下 [顯示進階 URL 設定]。

11. 在 [使用者屬性] 中，從 [使用者識別碼] 下拉式清單選取使用者的唯一識別碼。

12. **選擇性：** 按一下 [檢視和編輯所有其他使用者屬性]，以編輯當使用者登入時要以 SAML 權杖傳送至應用程式的屬性。

   若要新增屬性︰

   1. 按一下 [新增屬性]。 輸入 [名稱]，然後從下拉式清單選取 [值]。

   2. 按一下 [儲存]。 您會在資料表中看到新屬性。

13. 按一下 [設定 &lt;應用程式名稱&gt;]，以存取如何在應用程式中設定單一登入的文件。 此外，您會有透過該應用程式設定 SSO 所需的中繼資料 URL 與憑證。

14. 按一下 [儲存] 以儲存組態。

15. 將使用者指派至應用程式。

#### <a name="select-user-identifier-and-add-user-attributes-to-be-sent-to-the-application"></a>選取使用者識別碼並新增要傳送到應用程式的使用者屬性

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

   1. 按一下 [新增屬性]。 輸入 [名稱]，然後從下拉式清單選取 [值]。

   2. 按一下 [儲存]。 您會在資料表中看到新屬性。

#### <a name="download-the-azure-ad-metadata-or-certificate"></a>下載 Azure AD 中繼資料或憑證

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

### <a name="how-to-configure-federated-single-sign-on-for-a-non-gallery-application"></a>如何為不在資源庫內的應用程式設定同盟單一登入

若要設定不在資源庫內的應用程式，您必須有 Azure AD Premium，且應用程式必須支援 SAML 2.0。 如需有關 Azure AD 版本的詳細資訊，請參閱 [Azure AD 定價](https://azure.microsoft.com/pricing/details/active-directory/)。

-   [在 Azure AD 中設定應用程式的中繼資料值 (登入 URL、識別碼、回覆 URL)](#configuring-single-sign-on)

-   [選取使用者識別碼並新增要傳送到應用程式的使用者屬性](#select-user-identifier-and-add-user-attributes-to-be-sent-to-the-application)

-   [擷取 Azure AD 中繼資料與憑證](#download-the-azure-ad-metadata-or-certificate)

-   [在應用程式中設定 Azure AD 中繼資料值 (登入 URL、簽發者、登出 URL 與憑證)](#configuring-single-sign-on)

#### <a name="configure-the-applications-metadata-values-in-azure-ad-sign-on-url-identifier-reply-url"></a>在 Azure AD 中設定應用程式的中繼資料值 (登入 URL、識別碼、回覆 URL)

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

10. 在 [模式] 下拉式清單中選取 [SAML 登入]。

11. 在 [網域及 URL] 中輸入必要值 這些值應從應用程式廠商處取得。

   1. 若要將應用程式設定為 IdP 啟始的 SSO，請輸入回覆 URL 與識別碼。

   2.  **選擇性：** 若要將應用程式設定為 SP 啟始的 SSO，則登入 URL 為必要值。

12. 在 [使用者屬性] 中，從 [使用者識別碼] 下拉式清單選取使用者的唯一識別碼。

13. **選擇性：** 按一下 [檢視和編輯所有其他使用者屬性]，以編輯當使用者登入時要以 SAML 權杖傳送至應用程式的屬性。

   若要新增屬性︰

   1. 按一下 [新增屬性]。 輸入 [名稱]，然後從下拉式清單中選取 [值]。

   2. 按一下 [儲存]。 您會在資料表中看到新屬性。

14. 按一下 [設定 &lt;應用程式名稱&gt;]，以存取如何在應用程式中設定單一登入的文件。 此外，您有應用程式所需的 Azure AD URL 與憑證。

#### <a name="select-user-identifier-and-add-user-attributes-to-be-sent-to-the-application"></a>選取使用者識別碼並新增要傳送到應用程式的使用者屬性

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

#### <a name="download-the-azure-ad-metadata-or-certificate"></a>下載 Azure AD 中繼資料或憑證

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

### <a name="how-to-configure-password-single-sign-on-for-an-azure-ad-gallery-application"></a>如何為 Azure AD 資源庫應用程式設定密碼單一登入

若要設定 Azure AD 資源庫中的應用程式，您必須：

-   [從 Azure AD 資源庫新增應用程式](#add-an-application-from-the-azure-ad-gallery)

-   [設定應用程式使用密碼單一登入](#configure-the-application-for-password-single-sign-on)

#### <a name="add-an-application-from-the-azure-ad-gallery"></a>從 Azure AD 資源庫新增應用程式

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

#### <a name="configure-the-application-for-password-single-sign-on"></a>設定應用程式使用密碼單一登入

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

9.  [將使用者指派至應用程式](#how-to-assign-a-user-to-an-application-directly)。

10. 此外，您也可以選取使用者資料列，按一下 [更新認證]，然後代表使用者輸入使用者名稱和密碼，以代表使用者提供認證。 否則，系統會提示使用者在啟動時自行輸入認證。

### <a name="how-to-configure-password-single-sign-on-for-a-non-gallery-application"></a>如何為不在資源庫內的應用程式設定密碼單一登入

若要設定 Azure AD 資源庫中的應用程式，您必須：

-   [新增不在資源庫內的應用程式](#add-a-non-gallery-application)

-   [設定應用程式使用密碼單一登入](#configure-the-application-for-password-single-sign-on)

#### <a name="add-a-non-gallery-application"></a>新增不在資源庫內的應用程式

若要從 Azure AD 資源庫新增應用程式，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [企業應用程式] 窗格右上角的 [新增] 按鈕。

6.  按一下 [不在資源庫內的應用程式]。

7.  在 [名稱] 文字方塊中輸入應用程式的名稱。 選取 [新增]。

稍候片刻，您便能看見應用程式的設定窗格。

#### <a name="configure-the-application-for-password-single-sign-on"></a>設定應用程式使用密碼單一登入

若要設定應用程式使用單一登入，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

    1.  若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您要設定單一登入的應用程式。

7.  應用程式載入後，按一下應用程式的左側導覽功能表中的 [單一登入]。

8.  選取 [以密碼為基礎的登入] 模式。

9.  輸入**登入 URL**。 這是使用者輸入使用者名稱和密碼來登入的 URL。 請確保在此 URL 看得到登入欄位。

10. [將使用者指派給應用程式](#how-to-assign-a-user-to-an-application-directly)。

11. 此外，您也可以選取使用者資料列，按一下 [更新認證]，然後代表使用者輸入使用者名稱和密碼，以代表使用者提供認證。 否則，系統會提示使用者在啟動時自行輸入認證。

## <a name="problems-related-to-assigning-applications-to-users"></a>將應用程式指派給使用者的相關問題

使用者可能因為未將他們指派給應用程式，而無法在其存取面板上看見該應用程式。 以下是一些檢查方法：

-   [檢查是否已將使用者指派給應用程式](#check-if-a-user-is-assigned-to-the-application)

-   [如何將使用者直接指派給應用程式](#how-to-assign-a-user-to-an-application-directly)

-   [檢查是否已為使用者指派應用程式相關的授權](#check-if-a-user-is-under-a-license-related-to-the-application)

-   如何[將授權指派給 Azure AD 中的使用者](#how-to-assign-a-user-a-license)

### <a name="check-if-a-user-is-assigned-to-the-application"></a>檢查是否已將使用者指派給應用程式

若要檢查是否已將使用者指派給應用程式，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

6.  **搜尋**相關應用程式的名稱。

7.  按一下 [使用者和群組]。

8.  檢查是否已將使用者指派給應用程式。

   * 如果沒有，請依照＜如何將使用者直接指派給應用程式＞中的步驟來執行這項操作。

### <a name="how-to-assign-a-user-to-an-application-directly"></a>如何將使用者直接指派給應用程式

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

### <a name="check-if-a-user-is-under-a-license-related-to-the-application"></a>檢查使用者是否已獲應用程式相關的授權

若要檢查使用者獲指派的授權，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [所有使用者]。

6.  **搜尋**您感興趣的使用者，**按一下資料列**選取該使用者。

7.  按一下 [授權]，查看使用者目前已指派的授權。

  * 如果已為使用者指派 Office 授權，這會讓第一方 Office 應用程式出現在使用者的存取面板上。

### <a name="how-to-assign-a-user-a-license"></a>如何為使用者指派授權 

若要指派授權至使用者，請依照下列步驟執行︰

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

11. 按一下 [指派] 按鈕，將這些授權指派給這位使用者。

## <a name="problems-related-to-assigning-applications-to-groups"></a>將應用程式指派給群組的相關問題

使用者可能因為屬於已被指派應用程式的群組，所以能在存取面板上看見該應用程式。 以下是一些檢查方法：

-   [檢查使用者的群組成員資格](#check-a-users-group-memberships)

-   [如何將應用程式直接指派給群組](#how-to-assign-an-application-to-a-group-directly)

-   [檢查使用者是否屬於指派給授權的群組](#check-if-a-user-is-part-of-group-assigned-to-a-license)

-   [如何將授權指派給群組](#how-to-assign-a-license-to-a-group)

### <a name="check-a-users-group-memberships"></a>檢查使用者的群組成員資格

若要檢查群組的成員資格，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [所有使用者]。

6.  **搜尋**您感興趣的使用者，然後**按一下資料列**加以選取。

7.  按一下 [群組]。

8.  檢查使用者是否屬於已指派給應用程式的群組。

  * 如果您想要從群組移除使用者，請**按一下群組的資料列**，然後選取 [刪除]。

### <a name="how-to-assign-an-application-to-a-group-directly"></a>如何將應用程式直接指派給群組

若要將一或多個群組直接指派給應用程式，請依照下列步驟執行：

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

10. 在 [依姓名或電子郵件地址搜尋] 搜尋方塊中，輸入您有興趣指派之群組的**完整群組名稱**。

11. 將滑鼠停留在清單中的**群組**上方，以顯示**核取方塊**。 按一下群組設定檔照片或標誌旁邊的核取方塊，將使用者新增至 [已選取] 清單。

12. **選擇性︰** 如果您想要**新增多個群組**，請在 [依姓名或電子郵件地址搜尋] 搜尋方塊中，輸入另一個**完整群組名稱**，然後按一下核取方塊，將此群組新增至 [已選取] 清單。

13. 當您完成選取群組時，按一下 [選取] 按鈕，將它們新增到要指派給應用程式的使用者和群組清單。

14. **選擇性︰** 按一下 [新增指派] 窗格中的 [選取角色] 選取器，以選取要指派給您已選取之群組的角色。

15. 按一下 [指派] 按鈕，將應用程式指派給選取的群組。

稍後片刻，您已選取的使用者便能在存取面板中啟動這些應用程式。

### <a name="check-if-a-user-is-part-of-group-assigned-to-a-license"></a>檢查使用者是否屬於指派給授權的群組

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [所有使用者]。

6.  **搜尋**您感興趣的使用者，然後**按一下資料列**加以選取。

7.  按一下 [群組]。

8.  按一下特定群組的資料列。

9.  按一下 [授權]，以查看已指派給群組的授權。

   * 如果已將群組指派給 Office 授權，這可能會讓某些第一方 Office 應用程式出現在使用者的存取面板上。

### <a name="how-to-assign-a-license-to-a-group"></a>如何將授權指派給群組

若要將授權指派給群組，請依照下列步驟執行：

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
>若要更快速執行此作業，請考慮暫時將授權直接指派給使用者。 
>
>

## <a name="next-steps"></a>後續步驟
[將新的使用者加入 Azure Active Directory](./../fundamentals/add-users-azure-active-directory.md)

