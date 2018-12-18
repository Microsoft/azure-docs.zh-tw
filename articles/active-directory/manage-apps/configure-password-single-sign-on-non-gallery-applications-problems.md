---
title: 為不在資源庫內的應用程式設定密碼單一登入時遇到的問題 | Microsoft Docs
description: 了解使用者在為不在資源庫內的自訂應用程式設定密碼單一登入 (這類應用程式不會列於 Azure AD 應用程式庫中) 時所面臨的常見問題
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
ms.openlocfilehash: 3675889e583fbe2bf949891c3d6b4d5f731e6ac1
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44355005"
---
# <a name="problem-configuring-password-single-sign-on-for-a-non-gallery-application"></a>為不在資源庫內的應用程式設定密碼單一登入時遇到的問題

本文可協助您了解使用者在搭配不在資源庫內的應用程式設定**密碼單一登入**時所面臨的常見問題。

## <a name="how-to-capture-sign-in-fields-for-an-application"></a>如何擷取應用程式的登入欄位

只有具備 HTML 功能的登入頁面支援登入欄位擷取，而**非標準的登入頁面並不支援**，例如，使用快閃記憶體或其他不具備 HTML 功能技術的頁面。

有兩種方式可讓您用來擷取自訂應用程式的登入欄位：

-   自動登入欄位擷取

-   手動登入欄位擷取

**自動登入欄位擷取**適用於大部分具備 HTML 功能的登入頁面，但前提是這些頁面會**針對使用者名稱和密碼輸入欄位使用已知的 DIV 識別碼**。 其運作方式是藉由消除頁面上的 HTML 來尋找符合特定準則的 DIV 識別碼，然後儲存此應用程式的中繼資料，以便能在稍後重新執行密碼給它。

**手動登入欄位擷取**的適用情況是，應用程式**廠商並未標示**用來登入的輸入欄位。 手動登入欄位擷取也可適用於下列情況：當**廠商呈現多個無法自動偵測的欄位**時。 Azure AD 能在登入頁面上盡可能儲存最多欄位的資料，只要您告知我們那些欄位在頁面上的位置即可。

一般而言，**如果自動登入欄位擷取無法運作，請嘗試手動選項。**

### <a name="how-to-automatically-capture-sign-in-fields-for-an-application"></a>如何自動擷取應用程式的登入欄位

若要為使用**自動登入欄位擷取**的應用程式設定**密碼單一登入**，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

  * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您要設定單一登入的應用程式。

7.  應用程式載入後，按一下應用程式的左側導覽功能表中的 [單一登入]。

8.  選取 [以密碼為基礎的登入] 模式。

9.  輸入 [登入 URL]，使用者將在該處輸入其使用者名稱和密碼進行登入。 **請確定在您提供的 URL 看得到登入欄位**。

10. 按一下 [儲存]  按鈕。

11. 這麼做之後，該 URL 會自動擷取使用者名稱和密碼輸入方塊，並可讓您使用 Azure AD 並利用存取面板瀏覽器擴充功能將密碼安全地傳輸到該應用程式。

## <a name="how-to-manually-capture-sign-in-fields-for-an-application"></a>如何手動擷取應用程式的登入欄位

若要手動擷取登入欄位，您必須先安裝存取面板的瀏覽器延伸模組，而且**不能在 InPrivate、incognito 或私人模式中執行**。 若要安裝瀏覽器延伸模組，請依照[如何安裝存取面板的瀏覽器延伸模組](#i-cannot-manually-detect-sign-in-fields-for-my-application)一節中的步驟執行。

若要為使用**手動登入欄位擷取**的應用程式設定**密碼單一登入**，請依照下列步驟執行：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

   * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您要設定單一登入的應用程式。

7.  應用程式載入後，按一下應用程式的左側導覽功能表中的 [單一登入]。

8.  選取 [以密碼為基礎的登入] 模式。

9.  輸入 [登入 URL]，使用者將在該處輸入其使用者名稱和密碼進行登入。 **請確定在您提供的 URL 看得到登入欄位**。

10. 按一下 [儲存]  按鈕。

11. 這麼做之後，該 URL 會自動擷取使用者名稱和密碼輸入方塊，並可讓您使用 Azure AD 並利用存取面板瀏覽器擴充功能將密碼安全地傳輸到該應用程式。 萬一失敗，您可以藉由繼續執行步驟 12 來**變更登入模式以使用手動登入欄位擷取**。

12. 按一下 [設定 &lt;appname&gt; 密碼單一登入設定]。

13. 選取 [手動偵測登入欄位] 設定選項。

14. 按一下 [確定] 。

15. 按一下 [檔案] 。

16. 依照畫面指示來使用存取面板。

## <a name="i-see-a-we-couldnt-find-any-sign-in-fields-at-that-url-error"></a>我看到「在該 URL 找不到任何登入欄位」的錯誤

當自動偵測登入欄位失敗時，您就會看到此錯誤。 若要解決此問題，請依照[如何手動擷取應用程式的登入欄位](#how-to-manually-capture-sign-in-fields-for-an-application)一節中的步驟，嘗試執行手動登入欄位偵測。

## <a name="i-see-an-unable-to-save-single-sign-on-configuration-error"></a>我看到「無法儲存單一登入設定」錯誤

在某些罕見的情況下，更新單一登入設定可能會失敗。 若要解決此錯誤，請再次嘗試儲存單一登入設定。

如果持續不斷失敗，請開啟一個支援案例，並提供[如何查看入口網站通知的詳細資料](#i-cannot-manually-detect-sign-in-fields-for-my-application)和[如何將通知詳細資料傳送給支援工程師以取得協助](#how-to-get-help-by-sending-notification-details-to-a-support-engineer)小節中所收集的資訊。

## <a name="i-cannot-manually-detect-sign-in-fields-for-my-application"></a>我無法手動偵測應用程式的登入欄位

當手動偵測無法運作時，您可能看到某些行為，包括：

-   手動擷取程序看似正常運作，但擷取的欄位不正確

-   在執行擷取程序時並未反白顯示適當的欄位

-   擷取程序如預期般將我帶到應用程式的登入頁面，但什麼事也沒發生

-   手動擷取看似正常運作，但是，當我的使用者從存取面板瀏覽至應用程式，SSO 不會發生。

如果您遇到這其中任一個問題，請檢查下列內容：

-   確定您已依照[如何安裝存取面板的瀏覽器延伸模組](#how-to-install-the-access-panel-browser-extension)一節中的步驟，**安裝**並**啟用**最新版本的存取面板瀏覽器延伸模組。

-   確定您未在瀏覽器處於 **incognito、InPrivate 或私人模式**時嘗試擷取程序。 這些模式不支援存取面板延伸模組。

-   確定您的使用者未在 **incognito、InPrivate 或私人模式**中，嘗試從存取面板登入應用程式。 這些模式不支援存取面板延伸模組。

-   再次嘗試手動擷取程序，確保正確的欄位上都有紅色標記。

-   如果手動擷取程序看起來沒有回應，或者登入頁面不會執行任何動作 (上述的案例 3)，請再試一次手動擷取程序。 但這次已完成此程序，請按 **F12** 按鈕，以開啟瀏覽器的開發人員主控台。 此時，開啟 [主控台]並輸入 **window.location="&lt;輸入您在設定應用程式時所指定的登入 url&gt;"**，然後按 **Enter** 鍵。 這樣會強制執行頁面重新導向，以結束擷取程序並儲存已擷取的欄位。

如果這其中沒有任何一種方式適合您，客戶支援很樂意提供協助。 請開啟一個支援案例，並提供您所嘗試動作的詳細資料，以及[如何查看入口網站通知的詳細資料](#i-cannot-manually-detect-sign-in-fields-for-my-application)和[如何將通知詳細資料傳送給支援工程師以取得協助](#how-to-get-help-by-sending-notification-details-to-a-support-engineer)小節 (如果適用) 中所收集的資訊。

## <a name="how-to-install-the-access-panel-browser-extension"></a>如何安裝存取面板的瀏覽器延伸模組

若要安裝存取面板的瀏覽器延伸模組，請遵循下列步驟：

1.  在其中一種支援的瀏覽器中開啟[存取面板](https://myapps.microsoft.com)，然後在您的 Azure AD 中以**使用者**身分登入。

2.  按一下存取面板中的 [密碼-SSO 應用程式]。

3.  在要求安裝軟體的提示中，選取 [立即安裝]。

4.  系統會根據您的瀏覽器將您導向至下載連結。 將延伸模組**新增**到瀏覽器中。

5.  如果您的瀏覽器要求，請選取 [啟用] 或 [允許] 該延伸模組。

6.  安裝之後，**重新啟動**瀏覽器工作階段。

7.  登入存取面板，並查看您是否可以**啟動**密碼-SSO 應用程式。

您可能也會從下列直接連結中下載適用於 Chrome 和 Firefox 的延伸模組：

-   [Chrome 存取面板延伸模組](https://chrome.google.com/webstore/detail/access-panel-extension/ggjhpefgjjfobnfoldnjipclpcfbgbhl)

-   [Firefox 存取面板延伸模組](https://addons.mozilla.org/firefox/addon/access-panel-extension/)

## <a name="how-to-see-the-details-of-a-portal-notification"></a>如何查看入口網站通知的詳細資料

您可以依照下列步驟來查看任何入口網站通知的詳細資料：

1.  按一下 Azure 入口網站右上方的 [通知] 圖示 (鐘)。

2.  選取處於**錯誤**狀態的任何通知 (旁邊有紅色 (!))。

  >!NOTE] 您無法按一下處於**成功**或**進行中**狀態的通知。
  >
  >

3.  [通知詳細資料] 窗格隨即開啟。

4.  請自行利用這些資訊來了解更多問題的詳細資料。

5.  如果您仍然需要協助，您也可以將這些資訊分享給支援工程師或產品群組，以取得協助來解決您的問題。

6.  按一下 [複製錯誤] 文字方塊右邊的**複製****圖示**，以複製所有通知詳細資料來分享給支援工程師或產品群組工程師。

## <a name="how-to-get-help-by-sending-notification-details-to-a-support-engineer"></a>如何將通知詳細資料傳送給支援工程師以取得協助

如果您需要協助，一定要與支援工程師分享**下列所有詳細資料**，好讓他們儘快幫助您。 您可以**擷取螢幕畫面**，或按一下 [複製錯誤] 文字方塊右邊的**複製錯誤圖示**。

## <a name="notification-details-explained"></a>說明通知詳細資料

以下詳細說明每個通知項目的意義，並提供個別的範例。

### <a name="essential-notification-items"></a>必要通知項目

-   **標題** - 通知的描述性標題

    -   範例 - **應用程式 Proxy 設定**

-   **描述** - 作業所產生之結果的描述

    -   範例 - **輸入的內部 URL 正由另一個應用程式使用中**

-   **通知識別碼** – 通知的唯一識別碼

    -   範例 - **clientNotification-2adbfc06-2073-4678-a69f-7eb78d96b068**

-   **用戶端要求識別碼** – 瀏覽器所產生的特定要求識別碼

    -   範例 - **302fd775-3329-4670-a9f3-bea37004f0bc**

-   **時間戳記 UTC** - 發生通知期間的時間戳記 (UTC)

    -   範例 - **2017-03-23T19:50:43.7583681Z**

-   **內部交易識別碼** – 可用來在系統中查閱錯誤的內部識別碼

    -   範例 - **71a2f329-ca29-402f-aa72-bc00a7aca603**

-   **UPN** - 執行作業的使用者

    -   範例 - **tperkins@f128.info**

-   **租用戶識別碼** – 作業執行使用者所屬之租用戶的唯一識別碼

    -   範例 - **7918d4b5-0442-4a97-be2d-36f9f9962ece**

-   **使用者物件識別碼** – 執行作業之使用者的唯一識別碼

    -   範例 - **17f84be4-51f8-483a-b533-383791227a99**

### <a name="detailed-notification-items"></a>詳細通知項目

-   **顯示名稱** - **(可以是空白)** 錯誤的更詳細顯示名稱

    -   範例 *- **應用程式 Proxy 設定**

-   **狀態** - 通知的特定狀態

    -   範例 *- **失敗**

-   **物件識別碼** – **(可以是空白)** 據以執行作業的物件識別碼

    -   範例 - **8e08161d-f2fd-40ad-a34a-a9632d6bb599**

-   **詳細資料** - 作業所產生之結果的詳細描述

    -   範例 – **內部 url 'http://bing.com/' 無效，因為已在使用中**

-   **複製錯誤** - 按一下 [複製錯誤] 文字方塊右邊的**複製圖示**，以複製所有通知詳細資料來分享給支援工程師或產品群組工程師

    -   範例 - ```{"errorCode":"InternalUrl\_Duplicate","localizedErrorDetails":{"errorDetail":"Internal url 'http://google.com/' is invalid since it is already in use"},"operationResults":\[{"objectId":null,"displayName":null,"status":0,"details":"Internal url 'http://bing.com/' is invalid since it is already in use"}\],"timeStampUtc":"2017-03-23T19:50:26.465743Z","clientRequestId":"302fd775-3329-4670-a9f3-bea37004f0bb","internalTransactionId":"ea5b5475-03b9-4f08-8e95-bbb11289ab65","upn":"tperkins@f128.info","tenantId":"7918d4b5-0442-4a97-be2d-36f9f9962ece","userObjectId":"17f84be4-51f8-483a-b533-383791227a99"}```

## <a name="next-steps"></a>後續步驟
[使用應用程式 Proxy 提供單一登入應用程式](application-proxy-configure-single-sign-on-with-kcd.md)

