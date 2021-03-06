---
title: 教學課程：Azure Active Directory 單一登入 (SSO) 與 RedBrick Health 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 RedBrick Health 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/09/2019
ms.author: jeedes
ms.openlocfilehash: 9612a45d61e2bab03516699f661e8404fef6b7d1
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2020
ms.locfileid: "92515076"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-redbrick-health"></a>教學課程：Azure Active Directory 單一登入 (SSO) 與 RedBrick Health 整合

在本教學課程中，您將了解如何整合 RedBrick Health 與 Azure Active Directory (Azure AD)。 在整合 RedBrick Health 與 Azure AD 時，您可以︰

* 在 Azure AD 中控制可存取 RedBrick Health 的人員。
* 讓使用者使用其 Azure AD 帳戶自動登入 RedBrick Health。
* 在 Azure 入口網站集中管理您的帳戶。

若要深入了解 SaaS 應用程式與 Azure AD 整合，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>Prerequisites

若要開始，您需要下列項目：

* Azure AD 訂用帳戶。 如果沒有訂用帳戶，您可以取得[免費帳戶](https://azure.microsoft.com/free/)。
* 已啟用 RedBrick Health 單一登入 (SSO) 的訂用帳戶。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD SSO。

* RedBrick Health 支援由 **IDP** 起始的 SSO

> [!NOTE]
> 此應用程式的識別碼是固定的字串值，因此一個租用戶中只能設定一個執行個體。

## <a name="adding-redbrick-health-from-the-gallery"></a>從資源庫新增 RedBrick Health

若要設定將 RedBrick Health 整合到 Azure AD 中，您需要從資源庫將 RedBrick Health 新增到受控 SaaS 應用程式清單。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 [Azure 入口網站](https://portal.azure.com)。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory]  服務。
1. 巡覽至 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 若要新增應用程式，請選取 [新增應用程式]  。
1. 在 [從資源庫新增]  區段的搜尋方塊中輸入 **RedBrick Health** 。
1. 從結果面板選取 [RedBrick Health]  ，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。

## <a name="configure-and-test-azure-ad-single-sign-on-for-redbrick-health"></a>設定及測試 RedBrick Health 的 Azure AD 單一登入

以名為 **B.Simon** 的測試使用者，設定及測試與 RedBrick Health 搭配運作的 Azure AD SSO。 若要讓 SSO 能夠運作，您必須建立 Azure AD 使用者與 RedBrick Health 中相關使用者之間的連結關聯性。

若要設定及測試與 RedBrick Health 搭配運作的 Azure AD SSO，請完成下列建置組塊：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** - 讓您的使用者能夠使用此功能。
    1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 B.Simon 測試 Azure AD 單一登入。
    1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 B.Simon 能夠使用 Azure AD 單一登入。
1. **[設定 RedBrick Health SSO](#configure-redbrick-health-sso)** - 在應用程式端設定單一登入設定。
    1. **[建立 RedBrick Health 測試使用者](#create-redbrick-health-test-user)** - 使 RedBrick Health 中對應的 B.Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[測試 SSO](#test-sso)** - 驗證組態是否能運作。

## <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 [Azure 入口網站](https://portal.azure.com/)的 [RedBrick Health]  應用程式整合頁面上，尋找 [管理]  區段並選取 [單一登入]  。
1. 在 [ **選取單一登入方法** ] 頁面上，選取 [ **SAML** ]。
1. 在 [以 SAML 設定單一登入]  頁面上，按一下 [基本 SAML 設定]  的編輯/畫筆圖示，以編輯設定。

   ![編輯基本 SAML 組態](common/edit-urls.png)

1. 在 [基本 SAML 組態]  區段上，輸入下列欄位的值：

    a. 在 [識別碼]  文字方塊中，鍵入 URL：`https://www.redbrickhealth.com`

    b. 在 [回覆 URL]  文字方塊中，鍵入 URL：`https://sso-intg.redbrickhealth.com/sp/ACS.saml2`

    生產環境：`https://sso.redbrickhealth.com/sp/ACS.saml2`

    c. 按一下 [設定額外的 URL]  。

    d. 在 [轉送狀態]  文字方塊中，使用下列模式輸入 URL：`https://api-sso2.redbricktest.com/identity/sso/nbound?target=https://vanity9-sso2.redbrickdev.com/portal&connection=<companyname>conn1`

    > [!NOTE]
    > 轉送狀態的值不是真正的值。 請將這個值更新為實際的「轉送狀態」。 請連絡 [RedBrick Health 用戶端支援小組](https://home.redbrickhealth.com/contact/)以取得此值。 您也可以參考 Azure 入口網站中 **基本 SAML 組態** 區段所示的模式。

1. RedBrick Health 應用程式需要特定格式的 SAML 判斷提示。 設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性]  區段中，管理這些屬性的值。 在 [以 SAML 設定單一登入]  頁面上，按一下 [編輯]  按鈕以開啟 [使用者屬性]  對話方塊。

    ![顯示使用者屬性的螢幕擷取畫面，其中已選取 [編輯] 圖示。](common/edit-attribute.png)

1. 在 [使用者屬性]  對話方塊的 [使用者宣告]  區段中，如上圖所示設定 SAML 權杖屬性，然後執行下列步驟：

    | 名稱 | 來源屬性|
    | ----------- | --------- |
    | 主體名稱 | ********** |
    | 用戶端識別碼 | ********** |
    | 參與者識別碼 | ********** |

    > [!NOTE]
    > 這些值僅供參考之用。 您需要根據組織的需求定義屬性。 請連絡 [RedBrick Health 支援小組](https://home.redbrickhealth.com/contact/)，以取得必要宣告的詳細資訊。

    a. 按一下 [新增宣告]  以開啟 [管理使用者宣告]  對話方塊。

    ![顯示使用者宣告的螢幕擷取畫面，其中具有新增新宣告的選項。](common/new-save-attribute.png)

    ![顯示管理使用者宣告對話方塊的螢幕擷取畫面，您可以在其中輸入所述的值。](common/new-attribute-details.png)

    b. 在 [名稱]  文字方塊中，輸入該資料列所顯示的屬性名稱。

    c. 讓 [命名空間]  保持空白。

    d. 選取 [來源] 作為 [屬性]  。

    e. 在 [來源屬性]  清單中，輸入該資料列所顯示的屬性值。

    f. 按一下 [確定]  。

    g. 按一下 [檔案]  。

1. 在 [以 SAML 設定單一登入]  頁面的 [SAML 簽署憑證]  區段中，尋找 [憑證 (Base64)]  並選取 [下載]  ，以下載憑證並將其儲存在電腦上。

    ![憑證下載連結](common/certificatebase64.png)

1. 在 [設定 RedBrick Health]  區段上，依據您的需求複製適當的 URL。

    ![複製組態 URL](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

在本節中，您將在 Azure 入口網站中建立名為 B.Simon 的測試使用者。

1. 在 Azure 入口網站的左窗格中，依序選取 [Azure Active Directory]  、[使用者]  和 [所有使用者]  。
1. 在畫面頂端選取 [新增使用者]  。
1. 在 [使用者]  屬性中，執行下列步驟：
   1. 在 [名稱]  欄位中，輸入 `B.Simon`。  
   1. 在 [使用者名稱]  欄位中，輸入 username@companydomain.extension。 例如： `B.Simon@contoso.com` 。
   1. 選取 [顯示密碼]  核取方塊，然後記下 [密碼]  方塊中顯示的值。
   1. 按一下頁面底部的 [新增]  。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 RedBrick Health 的存取權授與 B. Simon，讓其能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 在應用程式清單中，選取 [RedBrick Health]  。
1. 在應用程式的概觀頁面中尋找 [管理]  區段，然後選取 [使用者和群組]  。

   ![[使用者和群組] 連結](common/users-groups-blade.png)

1. 選取 [新增使用者]  ，然後在 [新增指派]  對話方塊中選取 [使用者和群組]  。

    ![[新增使用者] 連結](common/add-assign-user.png)

1. 在 [使用者和群組]  對話方塊的 [使用者] 清單中選取 [B.Simon]  ，然後按一下畫面底部的 [選取]  按鈕。
1. 如果您在 SAML 判斷提示中需要任何角色值，請在 [選取角色]  對話方塊的清單中為使用者選取適當的角色，然後按一下畫面底部的 [選取]  按鈕。
1. 在 [新增指派]  對話方塊中，按一下 [指派]  按鈕。

## <a name="configure-redbrick-health-sso"></a>設定 RedBrick Health SSO

若要設定 **RedBrick Health** 端的單一登入，您必須將從 Azure 入口網站下載的 [憑證 (Base64)]  和複製的適當 URL 傳送給 [RedBrick Health 支援小組](https://home.redbrickhealth.com/contact/)。 他們會進行此設定，讓兩端的 SAML SSO 連線都設定正確。

### <a name="create-redbrick-health-test-user"></a>建立 RedBrick Health 測試使用者

在本節中，您會在 RedBrick Health 中建立名為 B.Simon 的使用者。 請與 [RedBrick Health 支援小組](https://home.redbrickhealth.com/contact/)合作，在 RedBrick Health 平台中新增使用者。 您必須先建立和啟動使用者，然後才能使用單一登入。

## <a name="test-sso"></a>測試 SSO

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [RedBrick Health] 圖格時，應該會自動登入您已設定 SSO 的 RedBrick Health。 如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/my-apps-portal-end-user-access.md)。

## <a name="additional-resources"></a>其他資源

- [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](./tutorial-list.md)

- [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

- [什麼是 Azure Active Directory 中的條件式存取？](../conditional-access/overview.md)

- [嘗試搭配 Azure AD 使用 RedBrick Health](https://aad.portal.azure.com/)