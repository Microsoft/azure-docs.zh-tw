---
title: 如何為 Azure AD 資源庫應用程式設定密碼單一登入 | Microsoft Docs
description: 當應用程式已列在 Azure AD 應用程式庫時，如何為它設定以密碼為基礎的安全單一登入
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
ms.openlocfilehash: 6f1e8efb63f62152c183035e1bb6289fc5db56dd
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44355089"
---
# <a name="how-to-configure-password-single-sign-on-for-an-azure-ad-gallery-application"></a>如何為 Azure AD 資源庫應用程式設定密碼單一登入

當您從 [Azure AD 應用程式庫](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis#get-started-with-the-azure-ad-application-gallery)新增應用程式時，您可以選擇希望使用者如何登入該應用程式。 您可以隨時在 [Azure 入口網站](https://portal.azure.com/)中選取企業應用程式的 [單一登入] 瀏覽項目，以設定這個選項。

可供您使用的其中一個單一登入方法是[以密碼為基礎的單一登入](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis#how-does-single-sign-on-with-azure-active-directory-work)選項。 這是開始將應用程式快速整合至 Azure AD 的絕佳方法，可讓您︰

-   針對已經與 Azure AD 整合的應用程式，安全地儲存和重播使用者名稱和密碼，以啟用**使用者的單一登入**

-   針對需要使用者名稱和密碼以外的更多欄位才能登入的應用程式，**支援需要多個登入欄位的應用程式**

-   針對使用者在[應用程式存取面板](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)輸入認證時所看到的使用者名稱和密碼輸入欄位，**自訂標籤**

-   針對**使用者**在[應用程式存取面板](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)手動輸入的任何現有應用程式帳戶，允許他們提供自己的使用者名稱和密碼

-   允許**商務群組的成員**使用[自助應用程式存取](https://docs.microsoft.com/azure/active-directory/active-directory-self-service-application-access)功能，指定要指派給使用者的使用者名稱和密碼

-   允許**管理員**在[將使用者指派至應用程式](#assign-a-user-to-an-application-directly)時，使用更新憑證功能指定要指派給使用者的使用者名稱和密碼

-   允許**管理員**在[將群組指派至應用程式](#assign-an-application-to-a-group-directly)時，使用更新憑證功能指定一群人使用的共用使用者名稱或密碼

下一節說明如何為已在 [Azure AD 應用程式庫](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis#get-started-with-the-azure-ad-application-gallery)中的應用程式啟用[以密碼為基礎的單一登入](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis#how-does-single-sign-on-with-azure-active-directory-work)。

## <a name="overview-of-steps-required"></a>所需步驟的概觀
若要設定 Azure AD 資源庫中的應用程式，您必須：

-   [從 Azure AD 資源庫新增應用程式](#add-an-application-from-the-azure-ad-gallery)

-   [設定應用程式使用密碼單一登入](#configure-the-application-for-password-single-sign-on)

-   [將應用程式指派至使用者或群組](#assign-the-application-to-a-user-or-a-group)

    -   [將使用者直接指派至應用程式](#assign-a-user-to-an-application-directly)

    -   [將應用程式直接指派至群組](#assign-an-application-to-a-group-directly)

## <a name="add-an-application-from-the-azure-ad-gallery"></a>從 Azure AD 資源庫新增應用程式

若要從 Azure AD 資源庫新增應用程式，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [企業應用程式] 窗格右上角的 [新增] 按鈕。

6.  在 [從資源庫新增] 區段的 [輸入名稱] 文字方塊中，輸入應用程式名稱。

7.  選取您要設為單一登入的應用程式。

8.  新增應用程式之前，您可以從 [名稱] 文字方塊變更其名稱。

9.  按一下 [新增] 按鈕新增應用程式。

稍候片刻，您便能看見應用程式的設定窗格。

## <a name="configure-the-application-for-password-single-sign-on"></a>設定應用程式使用密碼單一登入

若要設定應用程式使用單一登入，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

  * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您要設定單一登入的應用程式

7.  應用程式載入後，按一下應用程式的左側導覽功能表中的 [單一登入]。

8.  選取 [以密碼為基礎的登入] 模式。

9.  [將使用者指派至應用程式](#assign-a-user-to-an-application-directly)。

10. 此外，您也可以選取使用者資料列，按一下 [更新認證]，然後代表使用者輸入使用者名稱和密碼，以代表使用者提供認證。 否則，系統會提示使用者在啟動時自行輸入認證。

## <a name="assign-a-user-to-an-application-directly"></a>將使用者直接指派至應用程式

若要直接將一或多個使用者指派給應用程式，請遵循下列步驟︰

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

## <a name="assign-an-application-to-a-group-directly"></a>將應用程式直接指派至群組

若要將一或多個群組直接指派給應用程式，請遵循下列步驟：

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

稍待片刻，您已選取的使用者便能在存取面板中啟動這些應用程式。

## <a name="next-steps"></a>後續步驟
[使用應用程式 Proxy 提供單一登入應用程式](application-proxy-configure-single-sign-on-with-kcd.md)
