---
title: 我的應用程式清單中有未預期的應用程式 | Microsoft Docs
description: 如何檢視租用戶中的所有應用程式，並了解應用程式如何出現在 [企業應用程式] 下的 [所有應用程式] 清單中。
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
ms.date: 07/11/2017
ms.author: barbkess
ms.openlocfilehash: 754a06cc8bb9cc6e660e2ac5f01ef2c40176eb05
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44354993"
---
# <a name="unexpected-application-in-my-applications-list"></a>我的應用程式清單中有未預期的應用程式

本文協助您了解應用程式如何出現在 [企業應用程式] 下的 [所有應用程式] 清單中。 

## <a name="how-to-see-all-applications-in-your-tenant"></a>如何查看租用戶中的所有應用程式

若要查看租用戶中的所有應用程式，您需要使用 [篩選] 控制項，以顯示 [所有應用程式] 清單下的 [所有應用程式]。 請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

6.  按一下使用 [所有應用程式清單] 頂端的 [篩選]控制項。

7.  在 [篩選] 窗格上，將 [顯示] 選項設為 [所有應用程式]。

## <a name="why-does-a-specific-application-appear-in-my-all-applications-list"></a>特定應用程式為何出現在我的所有應用程式清單中？

篩選為 [所有應用程式] 時，[所有應用程式清單] 會顯示租用戶中的每個服務主體物件。 服務主體物件可能以各種方式出現在此清單中︰

1.  當您從應用程式庫新增任何應用程式時，包括︰

   1. **Azure AD 資源庫應用程式** – 與 Azure AD 預先整合以啟用單一登入的應用程式

   2. **應用程式 Proxy 應用程式** – 執行於您的內部部署環境中，而您想要讓使用者能夠安全地從外部對其進行單一登入的應用程式

   3. **自訂開發的應用程式** – 您的組織想要在 Azure AD 應用程式開發平台上開發的應用程式，但可能尚不存在

   4. **不在資源庫內的應用程式** – 引進您自己的應用程式！ 您想要的任何網頁連結，或任何呈現使用者名稱和密碼欄位的應用程式，都支援 SAML 或 OpenID Connect 通訊協定，或支援您想要與 Azure AD 整合以啟用單一登入的 SCIM。

2.  註冊或登入與 Azure Active Directory 整合的第三方<sup></sup>應用程式時。 其中一個範例是 [Smartsheet](https://app.smartsheet.com/b/home) 或 [DocuSign](https://www.docusign.net/member/MemberLogin.aspx)。

3.  註冊或將使用者或群組的授權新增至第一方應用程式時，例如 [Microsoft Office 365](http://products.office.com/)

4.  當您使用[應用程式登錄](https://docs.microsoft.com/azure/active-directory/active-directory-app-registration)建立自訂開發的應用程式來新增應用程式註冊時

5.  當您使用 [V2.0 應用程式註冊入口網站](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-app-registration#visit-the-microsoft-app-registration-portal)建立自訂開發的應用程式來新增應用程式註冊時

6.  當您新增您使用 Visual Studio 的 [ASP.net 驗證方法](http://www.asp.net/visual-studio/overview/2013/creating-web-projects-in-visual-studio#orgauthoptions)或[已連線服務](http://blogs.msdn.com/b/visualstudio/archive/2014/11/19/connecting-to-cloud-services.aspx)所開發的應用程式時

7.  當您使用 [Azure AD PowerShell 模組](/powershell/azure/install-adv2?view=azureadps-2.0)建立服務主體物件時

8.  當您以系統管理員身分[同意應用程式](https://docs.microsoft.com/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview#understanding-user-and-admin-consent)使用您租用戶中的資料時

9.  當[使用者同意應用程式](https://docs.microsoft.com/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview#understanding-user-and-admin-consent)使用您租用戶中的資料時

10. 當您啟用的某些服務會在您的租用戶中儲存資料時。 其中一個範例是「密碼重設」，它會模擬成服務主體，安全地儲存您的密碼重設原則。

若要取得如何將應用程式新增至目錄的詳細資訊，請參閱[如何及為何將應用程式新增至 Azure AD](https://docs.microsoft.com/azure/active-directory/develop/active-directory-how-applications-are-added)。

## <a name="i-want-to-remove-a-specific-users-or-groups-assignment-to-an-application"></a>我想要移除特定使用者或群組的應用程式指派

若要移除使用者或群組的應用程式指派，請依照[在 Azure Active Directory 中從企業應用程式移除使用者或群組指派](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-remove-assignment-azure-portal)文件列出的步驟執行。

## <a name="i-want-to-disable-all-access-to-an-application-for-every-user"></a>我想要讓每個使用者都無法存取應用程式

若要讓所有使用者都無法登入應用程式，請依照[在 Azure Active Directory 中停用使用者登入企業應用程式](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-disable-app-azure-portal)文件列出的步驟執行。

## <a name="i-want-to-delete-an-application-entirely"></a>我想要完全刪除應用程式

若要**刪除應用程式**，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

  * 若在這裡沒看到您要的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您要刪除的應用程式。

7.  應用程式載入後，在頂端應用程式的 [概觀] 窗格按一下 [刪除] 圖示。

## <a name="i-want-to-disable-all-future-user-consent-operations-to-any-application"></a>我想要停用任何應用程式的所有未來使用者同意作業

停用整個目錄的使用者同意會阻止使用者同意任何應用程式。 系統管理員仍然可以代表使用者行使同意。 若要進一步了解應用程式同意，以及您同意與否的可能原因，請參閱[了解使用者和系統管理員同意](https://docs.microsoft.com/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview#understanding-user-and-admin-consent)。

若要**停用整個目錄中的所有未來使用者同意作業**，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  按一下瀏覽功能表中的 [使用者和群組]。

5.  按一下 [使用者設定]。

6.  將 [使用者可以允許應用程式存取其資料] 切換開關設為 [否]，並按一下 [儲存] 按鈕，以停用所有未來的使用者同意作業。

## <a name="next-steps"></a>後續步驟
[使用 Azure Active Directory 管理應用程式](what-is-application-management.md)
