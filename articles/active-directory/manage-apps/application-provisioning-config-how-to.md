---
title: 如何設定 Azure AD 資源庫應用程式的使用者佈建 | Microsoft Docs
description: 如何為已經列於 Azure AD 應用程式庫中的應用程式快速設定豐富的使用者帳戶佈建和解除佈建
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
ms.reviewer: asteen
ms.openlocfilehash: ed2ace333b5bdf5210a1681f39dc60248117074e
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44355145"
---
# <a name="how-to-configure-user-provisioning-to-an-azure-ad-gallery-application"></a>如何設定 Azure AD 資源庫應用程式的使用者佈建

*使用者帳戶佈建*是指在應用程式的本機使用者設定檔儲存中建立、更新及/或停用使用者帳戶記錄的動作。 大部分的雲端和 SaaS 應用程式都會在自己的本機使用者設定檔儲存中儲存使用者角色和權限，而這類使用者記錄「必須」存在於其本機存放區中，才能讓單一登入和存取運作。

在 Azure 入口網站中，位於企業應用程式左側瀏覽窗格中的 [佈建] 索引標籤會顯示該應用程式支援哪些佈建模式。 這可能是下列其中一個值：

## <a name="configuring-an-application-for-manual-provisioning"></a>設定應用程式以進行手動佈建

「手動」佈建表示必須以手動方式使用該應用程式所提供的方法來建立使用者帳戶。 這可能表示登入該應用程式的系統管理員入口網站，然後使用 Web 架構使用者介面新增使用者。 或者，它可能會使用該應用程式所提供的機制，來上傳含有使用者帳戶詳細資料的試算表。 請參閱應用程式提供的文件，或洽詢應用程式開發人員以判斷有哪些機制可供使用。

如果「手動」是唯一針對指定應用程式所顯示的模式，這表示尚未針對該應用程式建立任何自動的 Azure AD 佈建連接器。 或者，這表示應用程式不支援必要條件的使用者管理 API，而您要在該 API 上建置自動化佈建連接器。

如果您想要求支援自動佈建指定的應用程式，可以使用 [Azure Active Directory 應用程式要求](https://aka.ms/aadapprequest)來填寫要求。

## <a name="configuring-an-application-for-automatic-provisioning"></a>設定應用程式以進行自動佈建

「自動」表示已經為此應用程式開發 Azure AD 佈建連接器。 如需 Azure AD 佈建服務及其運作方式的詳細資訊，請參閱[自動化使用 Azure Active Directory 對於 SaaS 應用程式的使用者佈建和解除佈建](https://docs.microsoft.com/azure/active-directory/active-directory-saas-app-provisioning)。

如需如何將特定使用者和群組佈建到應用程式的相關詳細資訊，請參閱[管理企業應用程式的使用者帳戶佈建](https://docs.microsoft.com/azure/active-directory/active-directory-enterprise-apps-manage-provisioning)。

啟用及設定自動佈建所需的實際步驟會因應用程式而有所不同。

>[!NOTE]
>您一開始應該先尋找設定您應用程式佈建專用的設定教學課程，然後依照這些步驟來設定應用程式和 Azure AD 以建立佈建連接。 
>
>

您可以在[如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)中找到應用程式教學課程。

設定佈建時必須考量的重點是，檢視和設定屬性 (Attribute) 對應，以及定義哪些使用者 (或群組) 屬性 (Property) 會從 Azure AD 流向應用程式的工作流程。 這包括設定「比對屬性」，此屬性可用於唯一識別並比對兩個系統之間的使用者/群組。 如需這個重要程序的詳細資訊。

## <a name="next-steps"></a>後續步驟
[在 Azure Active Directory 中自訂 SaaS 應用程式的使用者佈建屬性對應](https://docs.microsoft.com/azure/active-directory/active-directory-saas-customizing-attribute-mappings)

