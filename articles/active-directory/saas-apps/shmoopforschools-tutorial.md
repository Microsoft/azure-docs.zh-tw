---
title: 教學課程：Azure Active Directory 與 Shmoop For Schools 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Shmoop For Schools 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 1d75560a-55b3-42e9-bda1-92b01c572d8e
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/23/2018
ms.author: jeedes
ms.openlocfilehash: b5826fd3067ac337808b9e27040dee808cd6a01c
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39045952"
---
# <a name="tutorial-azure-active-directory-integration-with-shmoop-for-schools"></a>教學課程：Azure Active Directory 與 Shmoop For Schools 整合

在本教學課程中，您將了解如何整合 Shmoop For Schools 與 Azure Active Directory (Azure AD)。

Shmoop For Schools 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Shmoop For Schools 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Shmoop For Schools。
- 您可以在 Azure 入口網站集中管理您的帳戶。

如需 SaaS 應用程式與 Azure AD 整合的詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定與 Shmoop For Schools 的 Azure AD 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Shmoop For Schools 單一登入的訂用帳戶

> [!NOTE]
> 我們不建議使用生產環境來測試本教學課程中的步驟。

若要測試本教學課程中的步驟，我們建議：

- 只在必要時才使用生產環境。
- 如果您還沒有 Azure AD 試用環境，請取得[一個月免費試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二個主要建置組塊組成：

1. 從資源庫新增 Shmoop For Schools
2. 設定並測試 Azure AD 單一登入

## <a name="add-shmoop-for-schools-from-the-gallery"></a>從資源庫新增 Shmoop For Schools
若要設定將 Shmoop For Schools 整合到 Azure AD 中，您需要從資源庫將 Shmoop For Schools 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Shmoop For Schools，請採取下列步驟：**

1. 在 [Azure 入口網站](https://portal.azure.com)的左側窗格中，選取 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 移至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
3. 若要新增新的應用程式，請選取對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在 [搜尋] 方塊中，輸入 **Shmoop For Schools**。 從結果中選取 [Shmoop For Schools]，然後選取 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Shmoop For Schools](./media/shmoopforschools-tutorial/tutorial_shmoopforschools_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Shmoop For Schools 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Shmoop For Schools 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Shmoop For Schools 中的相關使用者之間建立連結關聯性。

若要使用 Shmoop For Schools 來設定並測試 Azure AD 單一登入，請完成下列建置組塊：

1. [設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)，讓您的使用者能夠使用此功能。
2. [建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)，以使用 Britta Simon 測試 Azure AD 單一登入。
3. [建立 Shmoop For Schools 測試使用者](#create-a-shmoop-for-schools-test-user) - 在 Shmoop For Schools 中建立 Britta Simon 的對應項目，且該項目須與 Azure AD 中代表使用者的項目連結。
4. [指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)，讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. [測試單一登入](#test-single-sign-on)，以驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Shmoop For Schools 應用程式中設定單一登入。

**若要使用 Shmoop For Schools 設定 Azure AD 單一登入功能，請執行下列步驟：**

1. 在 Azure 入口網站的 [Shmoop For Schools] 應用程式整合頁面上，選取 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊中，[單一登入模式] 下的下拉式功能表，選取 [SAML 型登入]。
 
    ![單一登入對話方塊](./media/shmoopforschools-tutorial/tutorial_shmoopforschools_samlbase.png)

3. 在 [Shmoop For Schools 網域與 URL] 區段中，採取下列步驟：

    ![設定單一登入](./media/shmoopforschools-tutorial/tutorial_shmoopforschools_url.png)

    a. 在 [登入 URL] 方塊中，以下列模式輸入 URL：`https://schools.shmoop.com/public-api/saml2/start/<uniqueid>`

    b. 在 [識別碼] 方塊中，以下列模式輸入 URL：`https://schools.shmoop.com/<uniqueid>`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [Shmoop For Schools 用戶端支援小組](mailto:support@shmoop.com)以取得這些值。 
 
4. Shmoop For Schools 應用程式需要特定格式的 SAML 判斷提示。 設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中管理這些屬性的值。 下列螢幕擷取畫面說明如何設定判斷提示：

    ![設定單一登入](./media/shmoopforschools-tutorial/tutorial_attribute.png)

    > [!NOTE]
    > Shmoop For Schools 支援兩個使用者角色：**老師**和**學生**。 在 Azure AD 中設定這些角色，以便您可以將使用者指派為適當的角色。 若要了解如何在 Azure AD 中設定角色，請參閱[使用 RBAC 和 Azure 入口網站來管理存取權](../../role-based-access-control/role-assignments-portal.md)。
    
5. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，設定 SAML 權杖屬性，如先前映像所示。  然後採取下列步驟：

    | 屬性名稱 | 屬性值 |
    | -------------- | --------------- |
    | 角色           | user.assignedroles |

    a. 若要開啟 [新增屬性] 對話方塊，請選取 [新增屬性]。
    
    ![設定單一登入 ](./media/shmoopforschools-tutorial/tutorial_attribute_04.png)
    
    ![設定單一登入](./media/shmoopforschools-tutorial/tutorial_attribute_05.png)
    
    b. 在 [名稱] 方塊中，輸入該資料列所顯示的屬性名稱。
    
    c. 在 [值] 清單中，選取該資料列所顯示的屬性值。

    d. 讓 [命名空間] 方塊保持空白。
    
    e. 選取 [確定]。

6. 選取 [儲存] 按鈕。

    ![設定單一登入](./media/shmoopforschools-tutorial/tutorial_general_400.png)

7. 在 [SAML 簽署憑證] 區段中，按一下「複製」按鈕複製「應用程式同盟中繼資料 URL」，並將它貼到 [記事本]。

    ![憑證下載連結](./media/shmoopforschools-tutorial/tutorial_shmoopforschools_certificate.png)

8. 若要在 **Shmoop For Schools** 端設定單一登入，您必須將**應用程式同盟中繼資料 URL** 傳送給 [Shmoop For Schools 支援小組](mailto:support@shmoop.com)。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 "Britta Simon" 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請採取下列步驟：**

1. 在 **Azure 入口網站**的左側窗格中，選取 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/shmoopforschools-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]。 然後選取 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/shmoopforschools-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，請選取 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/shmoopforschools-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，採取下列步驟：

    ![[使用者] 對話方塊](./media/shmoopforschools-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 選取 [建立] 。
 
### <a name="create-a-shmoop-for-schools-test-user"></a>建立 Shmoop For Schools 測試使用者

本節目標是在 Shmoop For Schools 中建立名為 Britta Simon 的使用者。 Shmoop For Schools 支援預設啟用的 Just-in-Time 佈建。 在這一節沒有您需要進行的動作項目。 當您嘗試存取 Shmoop For Schools 時，如果 Shmoop For Schools 還沒有使用者，它就會建立新的使用者。

>[!NOTE]
>如果您需要手動建立使用者，請連絡 [Shmoop For Schools 支援小組](mailto:support@shmoop.com)。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Shmoop For Schools 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Shmoop For Schools，請採取下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視。 然後移至目錄檢視中的**企業應用程式**。  接著，選取 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [Shmoop For Schools]。

    ![應用程式清單中的 Shmoop For Schools 連結](./media/shmoopforschools-tutorial/tutorial_shmoopforschools_app.png)  

3. 在左側功能表中，選取 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 選取 [新增] 按鈕。 然後，在 [新增指派] 對話方塊中，選取 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊中，選取 [使用者] 清單中的 [Britta Simon]。

6. 在 [使用者和群組] 對話方塊中，按一下 [選取] 按鈕。 

7. 在 [新增指派] 對話方塊中，選取 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入組態。

當您在存取面板中選取 [Shmoop For Schools] 圖格時，應該會自動登入您的 Shmoop For Schools 應用程式。

如需存取面板的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/shmoopforschools-tutorial/tutorial_general_01.png
[2]: ./media/shmoopforschools-tutorial/tutorial_general_02.png
[3]: ./media/shmoopforschools-tutorial/tutorial_general_03.png
[4]: ./media/shmoopforschools-tutorial/tutorial_general_04.png

[100]: ./media/shmoopforschools-tutorial/tutorial_general_100.png

[200]: ./media/shmoopforschools-tutorial/tutorial_general_200.png
[201]: ./media/shmoopforschools-tutorial/tutorial_general_201.png
[202]: ./media/shmoopforschools-tutorial/tutorial_general_202.png
[203]: ./media/shmoopforschools-tutorial/tutorial_general_203.png

