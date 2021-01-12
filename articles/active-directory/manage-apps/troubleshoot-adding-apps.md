---
title: 針對新增或移除應用程式以 Azure Active Directory 的常見問題進行疑難排解
description: 針對使用者新增或移除應用程式以 Azure Active Directory 時所面臨的常見問題進行疑難排解。
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 09/11/2018
ms.author: kenwith
ms.openlocfilehash: f873608d07785e6b99bd8e0fcdadf64a8899d490
ms.sourcegitcommit: 48e5379c373f8bd98bc6de439482248cd07ae883
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2021
ms.locfileid: "98108410"
---
# <a name="troubleshoot-common-problem-adding-or-removing-an-application-to-azure-active-directory"></a>針對新增或移除應用程式以 Azure Active Directory 的常見問題進行疑難排解
本文可協助您瞭解使用者在新增或移除應用程式以 Azure Active Directory 時所面臨的常見問題。

## <a name="i-clicked-the-add-button-and-my-application-took-a-long-time-to-appear"></a>按一下 [新增] 按鈕，但應用程式經過佷久才出現
在某些情況下，將應用程式新增至目錄後，可能需要 1-2 分鐘 (有時更久)，應用程式才會出現。 雖然這不是正常預期的表現，但只要透過按一下 [Azure 入口網站](https://portal.azure.com/)右上方的 [通知] 圖示 (鈴)，然後尋找標示為 [正在加入應用程式] 的 [進行中] 或 [已完成] 通知，即可看到應用程式新增作業正在進行中。

如果一直都沒有新增的應用程式，或您按一下 **[新增]** 按鈕時發生錯誤，您會看到 **錯誤** 狀態的 **通知**。 如果您想要取得更多關於此錯誤的詳細資料以進一步了解，或分享給支援工程師，則可以依照[如何查看入口網站通知的詳細資訊](#how-to-see-the-details-of-a-portal-notification)一節中的步驟，來查看此錯誤的詳細資訊。

## <a name="i-clicked-the-add-button-and-my-application-didnt-appear"></a>我按一下 [新增] 按鈕，但應用程式沒有出現
有時由於暫時性問題、網路問題或錯誤 (bug)，新增應用程式會失敗。 當您按一下 Azure 入口網站右上方的 [通知] 圖示 (鈴)，然後在 [正在加入應用程式] 通知旁邊看到紅色 (!) 圖示時，即可得知發生這種情況。 這表示建立應用程式時發生錯誤。

如果您按一下 [新增] 按鈕時發生錯誤，您會看到 **錯誤** 狀態的 **通知**。 如果您想要取得更多關於此錯誤的詳細資料以進一步了解，或分享給支援工程師，則可以依照[如何查看入口網站通知的詳細資訊](#how-to-see-the-details-of-a-portal-notification)一節中的步驟，來查看此錯誤的詳細資訊。

## <a name="i-dont-know-how-to-set-up-my-application-once-ive-added-it"></a>我在新增應用程式之後不知道如何設定
如果您需要瞭解應用程式的相關協助，請參閱 [如何整合 SaaS 應用程式與 Azure Active Directory](../saas-apps/tutorial-list.md) 文章的教學課程清單，是很好的開端。

此外，[Azure AD 應用程式文件庫](./what-is-application-management.md)也可幫助您進一步了解搭配 Azure AD 的單一登入及其運作方式。

## <a name="i-want-to-delete-an-application-but-the-delete-button-is-disabled"></a>我想要刪除應用程式，但 [刪除] 按鈕已停用

在下列案例中，將會停用 [刪除] 按鈕：

- 針對企業應用程式下的應用程式，如果您沒有下列其中一個角色：全域管理員、雲端應用程式系統管理員、應用程式系統管理員或服務主體的擁有者。

- 針對 Microsoft 應用程式，無論您的角色為何，都無法從 UI 中刪除它們。

- 適用于對應至受控識別的 servicePrincipals。 無法在企業應用程式分頁中刪除受控識別服務主體。 您必須移至 Azure 資源以進行管理。 深入瞭解 [受控識別](../managed-identities-azure-resources/overview.md)

## <a name="how-to-see-the-details-of-a-portal-notification"></a>如何查看入口網站通知的詳細資料
您可以依照下列步驟來查看任何入口網站通知的詳細資料：
1.  選取 [ **通知** ] 圖示 (Azure 入口網站右上方的 [鐘) ]
2.  選取處於 **錯誤** 狀態的任何通知 (旁邊有紅色 (!))。
    >[!NOTE]
    >您無法按一下 **成功** 或 **進行中** 狀態的通知。
4.  使用 [通知詳細資料] 中的資訊，來了解更多關於此問題的詳細資料。
5.  如果您仍然需要協助，您也可以將這項資訊分享給支援工程師或產品群組，以取得協助來解決您的問題。
6.  選取 [**複製錯誤**] 文字方塊右邊的 **複製圖示**，以複製所有通知詳細資料，以便與支援或產品群組工程師共用。

## <a name="how-to-get-help-by-sending-notification-details-to-a-support-engineer"></a>如何將通知詳細資料傳送給支援工程師以取得協助
如果您需要協助，請務必將 **以下列出的所有詳細資料** 與支援工程師分享，讓他們能夠快速地協助您。 **拍攝螢幕擷取畫面**，或選取 [**複製錯誤**] 文字方塊右邊的 [**複製錯誤] 圖示**。

## <a name="notification-details-explained"></a>說明通知詳細資料
如需更多關於通知的詳細資訊，請參閱下列說明。

### <a name="essential-notification-items"></a>必要通知項目
- **標題** - 通知的描述性標題
  * 範例 - **應用程式 Proxy 設定**
- **描述** - 作業所產生之結果的描述
  -   範例 - **輸入的內部 URL 正由另一個應用程式使用中**
- **通知識別碼** – 通知的唯一識別碼
  -   範例 - **clientNotification-2adbfc06-2073-4678-a69f-7eb78d96b068**
- **用戶端要求識別碼** – 瀏覽器所產生的特定要求識別碼
  -   範例 - **302fd775-3329-4670-a9f3-bea37004f0bc**
- **時間戳記 UTC** - 發生通知期間的時間戳記 (UTC)
  -   範例 - **2017-03-23T19:50:43.7583681Z**
- **內部交易識別碼** –我們可用來在系統中查閱錯誤的內部識別碼
  -   範例 - **71a2f329-ca29-402f-aa72-bc00a7aca603**
- **UPN** - 執行作業的使用者
  -   範例– **tperkins \@ f128.info**
- **租用戶識別碼** – 作業執行使用者所屬之租用戶的唯一識別碼
  -   範例 - **7918d4b5-0442-4a97-be2d-36f9f9962ece**
- **使用者物件識別碼** – 執行作業之使用者的唯一識別碼
  -   範例 - **17f84be4-51f8-483a-b533-383791227a99**

### <a name="detailed-notification-items"></a>詳細通知項目
-   **顯示名稱** - **(可以是空白)** 錯誤的更詳細顯示名稱
    -   範例 - **應用程式 Proxy 設定**
-   **狀態** - 通知的特定狀態
    -   範例– **失敗**
-   **物件識別碼** – **(可以是空白)** 據以執行作業的物件識別碼
    -   範例 - **8e08161d-f2fd-40ad-a34a-a9632d6bb599**
-   **詳細資料** - 作業所產生之結果的詳細描述
    -   範例– **內部 url `https://bing.com/` 無效，因為它已在使用中**
-   **複製錯誤**–選取 [**複製錯誤**] 文字方塊右邊的 **複製圖示**，以複製所有通知詳細資料，以便與支援或產品群組共用 
-   工程師
    -   範例 ```{"errorCode":"InternalUrl\_Duplicate","localizedErrorDetails":{"errorDetail":"Internal url 'https://google.com/' is invalid since it is already in use"},"operationResults":\[{"objectId":null,"displayName":null,"status":0,"details":"Internal url 'https://bing.com/' is invalid since it is already in use"}\],"timeStampUtc":"2017-03-23T19:50:26.465743Z","clientRequestId":"302fd775-3329-4670-a9f3-bea37004f0bb","internalTransactionId":"ea5b5475-03b9-4f08-8e95-bbb11289ab65","upn":"tperkins@f128.info","tenantId":"7918d4b5-0442-4a97-be2d-36f9f9962ece","userObjectId":"17f84be4-51f8-483a-b533-383791227a99"}```
