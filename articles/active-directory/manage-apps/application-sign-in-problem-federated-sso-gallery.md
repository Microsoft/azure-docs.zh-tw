---
title: 登入針對同盟單一登入設定之資源庫應用程式的問題 | Microsoft Docs
description: 當登入您已使用 Azure AD 針對 SAML 型同盟單一登入設定之應用程式時所發生特定錯誤的指引
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
ms.openlocfilehash: 8d910ffcf966e98def33a42a6452baea9f4b3998
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44355198"
---
# <a name="problems-signing-in-to-a-gallery-application-configured-for-federated-single-sign-on"></a>登入針對同盟單一登入設定之資源庫應用程式的問題

若要疑難排解您的問題，您必須在 Azure AD 中驗證應用程式組態，如下所示：

-   您已遵循 Azure AD 資源庫應用程式的所有設定步驟。

-   在 AAD 中設定的識別碼與回覆 URL 符合其在應用程式中的預期值

-   您已將使用者指派至應用程式

## <a name="application-not-found-in-directory"></a>在目錄中找不到應用程式

*錯誤 AADSTS70001：在目錄中找不到識別碼為 ‘ https://contoso.com’ 的應用程式*。

**可能的原因**

以 SAML 要求從應用程式傳送到 Azure AD 的簽發者要求屬性，不符合應用程式 Azure AD 中所設定的識別碼值。

**解決方案**

確保 SAML 要求中的簽發者屬性符合 Azure AD 中設定的識別碼值：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

  * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您要設定單一登入的應用程式

7.  應用程式載入後，按一下應用程式的左側導覽功能表中的 [單一登入]。

8.  移至 [網域及 URL] 區段。 確認 [識別碼] 文字方塊中的值符合錯誤中所顯示的識別碼值。

您已在 Azure AD 中更新識別碼值，且該值符合應用程式以 SAML 要求傳送的值後，您應該能夠登入應用程式。

## <a name="the-reply-address-does-not-match-the-reply-addresses-configured-for-the-application"></a>回覆位址不符合為應用程式設定的回覆位址。

*錯誤 AADSTS50011：回覆位址 ‘ https://contoso.com’ 不符合為應用程式設定的回覆位址*

**可能的原因**

SAML 要求中的 AssertionConsumerServiceURL 值不符合在 Azure AD 中設定的 [回覆 URL] 值或模式。 SAML 要求中的 AssertionConsumerServiceURL 值是您在錯誤中看到的 URL。

**解決方案**

確保 SAML 要求中的 AssertionConsumerServiceURL 值符合在 Azure AD 中設定的 [回覆 URL] 值。

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

  * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您要設定單一登入的應用程式

7.  應用程式載入後，按一下應用程式的左側導覽功能表中的 [單一登入]。

8.  移至 [網域及 URL] 區段。 驗證或更新 [回覆 URL] 文字方塊中的值，以符合 SAML 要求中的 AssertionConsumerServiceURL 值。  
    * 如果沒有看到 [回覆 URL] 文字方塊中，請選取 [顯示進階 URL 設定] 核取方塊。

您已在 Azure AD 中更新 [回覆 URL] 值，且該值符合應用程式以 SAML 要求傳送的值後，您應該能夠登入應用程式。

## <a name="user-not-assigned-a-role"></a>使用者未指派角色

*錯誤 AADSTS50105：未將已登入的使用者 'brian@contoso.com' 指派至應用程式的角色*。

**可能的原因**

尚未將 Azure AD 中應用程式的存取權授與使用者。

**解決方案**

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

稍待片刻，您已選取的使用者就能夠使用解決方案描述一節所述的方法，啟動這些應用程式。

## <a name="not-a-valid-saml-request"></a>非有效的 SAML 要求

*錯誤 AADSTS75005：要求不是有效的 Saml2 通訊協定訊息。*

**可能的原因**

Azure AD 不支援應用程式為單一登入傳送的 SAML 要求。 以下為一些常見問題：

-   SAML 要求中遺漏必要的欄位

-   SAML 要求編碼方法

**解決方案**

1.  擷取 SAML 要求。 依照[如何偵錯 SAML 型單一登入 Azure Active Directory 中的應用程式](https://docs.microsoft.com/azure/active-directory/develop/active-directory-saml-debugging)教學課程學習如何擷取 SAML 要求。

2.  請連絡應用程式廠商並告知︰

   -   SAML 要求

   -   [Azure AD 單一登入 SAML 通訊協定需求](https://docs.microsoft.com/azure/active-directory/develop/active-directory-single-sign-on-protocol-reference)

廠商應確認其支援單一登入的 AD SAML 實作。

## <a name="no-resource-in-requiredresourceaccess-list"></a>requiredResourceAccess 清單中沒有資源

*錯誤 AADSTS65005：用戶端應用程式已要求存取資源 '00000002-0000-0000-c000-000000000000'。此要求失敗，因為用戶端尚未在其 requiredResourceAccess 清單中指定此資源*。

**可能的原因**

應用程式物件已損毀。

**解決方式：選項 1**

若要解決此問題，請在 Azure AD 組態新增唯一的 [識別碼] 值。 若要新增 [識別碼] 值，請依照下列步驟執行︰

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

  * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您已設定單一登入的應用程式。

7.  應用程式載入後，按一下應用程式的左側瀏覽功能表中的 [單一登入]。

8.  在 [網域和 URL] 區段之下，勾選 [顯示進階 URL 設定]。

9.  在 [識別碼] 文字方塊中輸入應用程式的唯一識別碼。

10. **儲存**組態。


**解決方式：選項 2**

如果上述選項 1 無效，請試著從目錄中移除應用程式。 然後，依照下列步驟新增並重新設定應用程式：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

  * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您要設定單一登入的應用程式

7.  按一下應用程式 [概觀] 窗格左上方的 [刪除]。

8.  重新整理 Azure AD，並從 Azure AD 資源庫新增應用程式。 接著，設定應用程式

<span id="_Hlk477190176" class="anchor"></span>在重新設定應用程式後，您應該能夠登入應用程式。

## <a name="certificate-or-key-not-configured"></a>未設定憑證或金鑰

*錯誤 AADSTS50003︰未設定簽署金鑰。*

**可能的原因**

應用程式物件已損毀，Azure AD 無法辨識為應用程式設定的憑證。

**解決方案**

若要刪除與建立新的憑證，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

 * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您要設定單一登入的應用程式

7.  應用程式載入後，按一下應用程式的左側導覽功能表中的 [單一登入]。

8.  在 [SAML 簽署憑證] 區段下方，按一下 [建立新憑證]。

9.  選取到期日。 然後按一下 [儲存] 。

10. 勾選 [Make new certificate active (啟用新憑證)] 以覆寫作用中的憑證。 然後按一下窗格頂端的 [儲存]，並按 [接受] 以啟動變換憑證。

11. 在 [SAML 簽署憑證] 區段下，按一下 [移除] 以移除**未使用**的憑證。

## <a name="problem-when-customizing-the-saml-claims-sent-to-an-application"></a>自訂傳送至應用程式的 SAML 宣告時發生問題

若要了解如何自訂傳送至應用程式的 SAML 屬性宣告，請參閱 [Azure Active Directory 中的宣告對應](https://docs.microsoft.com/azure/active-directory/active-directory-claims-mapping)以取得詳細資訊。

## <a name="next-steps"></a>後續步驟
[如何偵錯 SAML 型單一登入 Azure Active Directory 中的應用程式](https://docs.microsoft.com/azure/active-directory/develop/active-directory-saml-debugging)
