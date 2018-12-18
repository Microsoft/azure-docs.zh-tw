---
title: 教學課程：Azure Active Directory 與 vxMaintain 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 vxMaintain 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 841a1066-593c-4603-9abe-f48496d73d10
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/26/2018
ms.author: jeedes
ms.openlocfilehash: 7e444692dfeab5ca14fbd896043cc28e2cbd8717
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39433989"
---
# <a name="tutorial-azure-active-directory-integration-with-vxmaintain"></a>教學課程：Azure Active Directory 與 vxMaintain 整合

在本教學課程中，您會了解如何將 vxMaintain 與 Azure Active Directory (Azure AD) 整合。

這項整合提供一些重要優勢。 您可以：

- 在 Azure AD 中控制可存取 vxMaintain 的人員。
- 可以讓您的使用者藉由使用其 Azure AD 帳戶，透過單一登入 (SSO) 自動登入 vxMaintain。
- 在 Azure 入口網站中集中管理您的帳戶。

若要深入了解 SaaS 應用程式與 Azure AD 整合，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 vxMaintain 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 vxMaintain SSO 的訂用帳戶

> [!NOTE]
> 當您測試本教學課程中的步驟時，不建議您使用生產環境。

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 

本教學課程中說明的案例由二個主要構成要素組成：

* 從資源庫新增 vxMaintain
* 設定並測試 Azure AD 單一登入

## <a name="add-vxmaintain-from-the-gallery"></a>從資源庫新增 vxMaintain
若要設定將 vxMaintain 與 Azure AD 整合，您需要從資源庫將 vxMaintain 新增到受控 SaaS 應用程式清單。

若要從資源庫新增 vxMaintain，請執行下列動作：

1. 在 [Azure 入口網站](https://portal.azure.com)的左側窗格中，選取 [Azure Active Directory] 按鈕。 

    ![Azure Active Directory 按鈕][1]

1. 選取 [企業應用程式] > [所有應用程式]。

    ![[企業應用程式] 窗格][2]
    
1. 若要新增應用程式，請在 [所有應用程式] 對話方塊中，選取 [新增應用程式]。

    ![[新增應用程式] 按鈕][3]

1. 在搜尋方塊中，輸入 **vxMaintain**。

    ![[單一登入模式] 下拉式清單](./media/vxmaintain-tutorial/tutorial_vxmaintain_search.png)

1. 在結果清單中，選取 [vxMaintain]，然後選取 [新增]。

    ![vxMaintain 連結](./media/vxmaintain-tutorial/tutorial_vxmaintain_addfromgallery.png)

##  <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，使用 vxMaintain 設定及測試 Azure AD SSO。

若要讓單一登入能夠運作，Azure AD 必須知道 vxMaintain 與 Azure AD 中互相對應的使用者。 也就是說，您必須在 Azure AD 使用者和對應的 vxMaintain 使用者之間建立連結關聯性。

若要建立連結關聯性，請將 vxMaintain **使用者名稱**值指派為 Azure AD **Username** 值。

若要使用 vxMaintain 設定及測試 Azure AD SSO，請完成下列構成要素。

### <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

在本節中，您可以藉由執行下列步驟，在 Azure 入口網站中啟用 Azure AD SSO，並在 vxMaintain 應用程式中設定 SSO：

1. 在 Azure 入口網站的 [vxMaintain] 應用程式整合頁面上，選取 [單一登入]。

    ![[單一登入] 命令][4]

1. 若要啟用 SSO，請在 [單一登入模式] 下拉式清單中，選取 [SAML 型登入]。
 
    ![[SAML 型登入] 命令](./media/vxmaintain-tutorial/tutorial_vxmaintain_samlbase.png)

1. 在 [vxMaintain 網域與 URL] 中，執行下列動作：

    ![[vxMaintain 網域與 URL] 區段](./media/vxmaintain-tutorial/tutorial_vxmaintain_url.png)

    a. 在 [識別碼] 方塊中，輸入具有下列語法的 URL：`https://<company name>.verisae.com`

    b. 在 [回覆 URL] 方塊中，輸入具有下列語法的 URL：`https://<company name>.verisae.com/DataNett/action/ssoConsume/mobile?_log=true`

    > [!NOTE] 
    > 上述值並非真正的值。 請使用實際的識別碼和回覆 URL 來更新這些值。 若要取得這些值，請連絡 [vxMaintain 支援小組](https://www.hubspot.com/company/contact)。
 
1. 在 [SAML 簽署憑證] 下，選取 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![[SAML 簽署憑證] 區段](./media/vxmaintain-tutorial/tutorial_vxmaintain_certificate.png) 

1. 選取 [ **儲存**]。

    ![[儲存] 按鈕](./media/vxmaintain-tutorial/tutorial_general_400.png)

1. 若要設定 **vxMaintain** SSO，將下載的**中繼資料 XML** 檔案傳送至 [vxMaintain 支援小組](https://www.hubspot.com/company/contact)。

> [!TIP]
> 當您設定應用程式時，您可以在 [Azure 入口網站](https://portal.azure.com)中閱讀精簡版的先前說明。 當您從 [Active Directory] > [企業應用程式] 區段新增應用程式之後，選取 [單一登入] 索引標籤，即可從 [設定] 區段存取內嵌的文件。 
>
>若要深入了解內嵌文件功能，請參閱[管理企業應用程式的單一登入](https://go.microsoft.com/fwlink/?linkid=845985)。
> 

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
在本節中，您會執行下列步驟，在 Azure 入口網站中建立測試使用者 Britta Simon：

![Azure AD 測試使用者][100]

1. 在 **Azure 入口網站**的左側窗格中，選取 [Azure Active Directory] 按鈕。

    ![[Azure Active Directory] 按鈕](./media/vxmaintain-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組] > [所有使用者]。
    
    ![[所有使用者] 連結](./media/vxmaintain-tutorial/create_aaduser_02.png)  
    [所有使用者] 對話方塊隨即開啟。 

1. 若要開啟 [使用者] 對話方塊，請選取 [新增]。
 
    ![[新增] 按鈕](./media/vxmaintain-tutorial/create_aaduser_03.png) 

1. 在 [使用者] 對話方塊中，執行下列動作：
 
    ![[使用者] 對話方塊](./media/vxmaintain-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入測試使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中產生的值。

    d. 選取 [建立] 。
 
### <a name="create-a-vxmaintain-test-user"></a>建立 vxMaintain 測試使用者

在本節中，您會在 vxMaintain 中建立測試使用者 Britta Simon。 若要在 vxMaintain 平台中新增使用者，請與 [vxMaintain 支援小組](https://www.hubspot.com/company/contact) 合作。 使用 SSO 之前，請先建立並啟用使用者。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會把 vxMaintain 的存取權授予 Britta Simon，讓 Britta Simon 能夠使用 Azure SSO。 若要這樣做，請執行下列動作：

![[顯示名稱] 清單中的測試使用者][200] 

1. 在 Azure 入口網站的 [應用程式] 檢視中，移至 [目錄] 檢視 > [企業應用程式] > [所有應用程式]。

    ![[所有應用程式] 連結][201] 

1. 在 [應用程式] 清單中，選取 [vxMaintain]。

    ![vxMaintain 連結](./media/vxmaintain-tutorial/tutorial_vxmaintain_app.png) 

1. 在左側窗格中，選取 [使用者和群組]。

    ![[使用者和群組] 連結][202] 

1. 選取 [新增]，然後在 [新增指派] 窗格中，選取 [使用者和群組]。

    ![[使用者和群組] 連結][203]

1. 在 [使用者和群組] 對話方塊的 [使用者] 清單中，選取 [Britta Simon]，然後選取 [選取] 按鈕。

1. 在 [新增指派] 對話方塊中，選取 [指派]。
    
### <a name="test-your-azure-ad-single-sign-on"></a>測試您的 Azure AD 單一登入

在本節中，您會使用存取面板來測試您的 Azure AD SSO 組態。

在存取面板中選取 [vxMaintain] 圖格時，應該會自動登入您的 vxMaintain 應用程式。

如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="next-steps"></a>後續步驟

* [整合 SaaS 應用程式與 Azure Active Directory 的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/vxmaintain-tutorial/tutorial_general_01.png
[2]: ./media/vxmaintain-tutorial/tutorial_general_02.png
[3]: ./media/vxmaintain-tutorial/tutorial_general_03.png
[4]: ./media/vxmaintain-tutorial/tutorial_general_04.png

[100]: ./media/vxmaintain-tutorial/tutorial_general_100.png

[200]: ./media/vxmaintain-tutorial/tutorial_general_200.png
[201]: ./media/vxmaintain-tutorial/tutorial_general_201.png
[202]: ./media/vxmaintain-tutorial/tutorial_general_202.png
[203]: ./media/vxmaintain-tutorial/tutorial_general_203.png

