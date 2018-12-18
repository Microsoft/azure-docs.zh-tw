---
title: 新增不在資源庫內的應用程式時遇到問題 | Microsoft Docs
description: 了解新增不在資源庫內的自訂應用程式時的常見問題
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
ms.date: 09/11/2018
ms.author: barbkess
ms.openlocfilehash: 01f213f0a02913c2d0b632542160b25c97fec989
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44713369"
---
# <a name="problem-adding-a-non-gallery-application"></a>新增不在資源庫內的應用程式時遇到問題

本文將協助您了解新增**不在資源庫內的自訂應用程式**時的常見問題及如何解決。 

## <a name="i-clicked-the-add-button-and-my-application-took-a-long-time-to-appear"></a>按一下 [新增] 按鈕，但應用程式經過佷久才出現

在某些情況下，將應用程式新增至目錄後，可能需要 1-2 分鐘 (有時更久)，應用程式才會出現。 雖然這不是正常預期的表現，您還是可以按一下 [Azure 入口網站](https://portal.azure.com/)右上角的 **通知**圖示 (鐘)，然後尋找標示為**建立應用程式**的**進行中**或**已完成**通知，來查看應用程式新增作業正在進行中。

如果一直都沒有新增的應用程式，或您按一下 **[新增]** 按鈕時發生錯誤，您會看到**錯誤**狀態的**通知**。 如果您想要取得更多關於此錯誤的詳細資料以進一步了解，或分享給支援工程師，則可以依照[如何查看入口網站通知的詳細資訊](#how-to-see-the-details-of-a-portal-notification)一節中的步驟，來查看此錯誤的詳細資訊。

## <a name="i-clicked-the-add-button-and-my-application-didnt-appear"></a>我按一下 [新增] 按鈕，但應用程式沒有出現

有時由於暫時性問題、網路問題或錯誤，新增應用程式會失敗。 當您按一下 Azure 入口網站右上方的**通知**圖示 (鐘)，然後在 [建立應用程式] 通知旁看到紅色 (!) 圖示時，就表示發生這種情況。 這表示建立應用程式時發生錯誤。

如果您按一下 [新增] 按鈕時發生錯誤，您會看到**錯誤**狀態的**通知**。 如果您想要取得更多關於此錯誤的詳細資料以進一步了解，或分享給支援工程師，則可以依照[如何查看入口網站通知的詳細資訊](#how-to-see-the-details-of-a-portal-notification)一節中的步驟，來查看此錯誤的詳細資訊。

## <a name="i-dont-know-how-to-set-up-my-application-once-ive-added-it"></a>我在新增應用程式之後不知道如何設定

如果您需要了解自訂應用程式，[Azure AD 應用程式文件庫](https://docs.microsoft.com/azure/active-directory/active-directory-apps-index)可幫助您進一步了解搭配 Azure AD 的單一登入和其運作方式。

## <a name="how-to-see-the-details-of-a-portal-notification"></a>如何查看入口網站通知的詳細資料

您可以依照下列步驟來查看任何入口網站通知的詳細資料：

1.  按一下 Azure 入口網站右上方的 [通知] 圖示 (鐘)。

2.  選取處於**錯誤**狀態的任何通知 (旁邊有紅色 (!))。

   >[!NOTE]
   >您無法按一下**成功**或**進行中**狀態的通知。
   >
   >

4.  使用 [通知詳細資料] 中的資訊，來了解更多關於此問題的詳細資料。

5.  如果您仍然需要協助，您也可以將這項資訊分享給支援工程師或產品群組，以取得協助來解決您的問題。

6.  按一下 [複製錯誤] 文字方塊右邊的**複製圖示**，以複製所有通知詳細資料來分享給支援工程師或產品群組工程師。

## <a name="how-to-get-help-by-sending-notification-details-to-a-support-engineer"></a>如何將通知詳細資料傳送給支援工程師以取得協助

如果您需要協助，一定要與支援工程師分享**下列所有詳細資料**，好讓他們儘快幫助您。 只要**擷取螢幕畫面**，或按一下 [複製錯誤] 文字方塊右邊的**複製錯誤圖示**，輕鬆就能這樣做。

## <a name="notification-details-explained"></a>說明通知詳細資料

如需更多關於通知的詳細資訊，請參閱下列說明。

### <a name="essential-notification-items"></a>必要通知項目

-   **標題** - 通知的描述性標題
   *  範例 - **應用程式 Proxy 設定**

-   **描述** - 作業所產生之結果的描述

   *  範例 - **輸入的內部 URL 正由另一個應用程式使用中**

-   **通知識別碼** – 通知的唯一識別碼

   *  範例 - **clientNotification-2adbfc06-2073-4678-a69f-7eb78d96b068**

-   **用戶端要求識別碼** – 瀏覽器所產生的特定要求識別碼

   *  範例 - **302fd775-3329-4670-a9f3-bea37004f0bc**

-   **時間戳記 UTC** - 發生通知期間的時間戳記 (UTC)

   *  範例 - **2017-03-23T19:50:43.7583681Z**

-   **內部交易識別碼** – 可用來在系統中查閱錯誤的內部識別碼

   *  範例 - **71a2f329-ca29-402f-aa72-bc00a7aca603**

-   **UPN** - 執行作業的使用者

   *  範例 - **tperkins@f128.info**

-   **租用戶識別碼** – 作業執行使用者所屬之租用戶的唯一識別碼

   *  範例 - **7918d4b5-0442-4a97-be2d-36f9f9962ece**

-   **使用者物件識別碼** – 執行作業之使用者的唯一識別碼

 *  範例 - **17f84be4-51f8-483a-b533-383791227a99**

### <a name="detailed-notification-items"></a>詳細通知項目

-   **顯示名稱** – **(可以是空白)** 錯誤的詳細顯示名稱

  *  範例 - **應用程式 Proxy 設定**

-   **狀態** – 通知的特定狀態

   *  範例 – **失敗**

-   **物件識別碼** – **(可以是空白)** 據以執行作業的物件識別碼

   *  範例 - **8e08161d-f2fd-40ad-a34a-a9632d6bb599**

-   **詳細資料** - 作業所產生之結果的詳細描述

   *  範例 – **內部 url 'http://bing.com/' 無效，因為已在使用中**

-   **複製錯誤** – 按一下 [複製錯誤] 文字方塊右邊的**複製圖示**，以複製所有通知詳細資料來分享給支援工程師或產品群組 
-   工程師

   *  範例 ```{"errorCode":"InternalUrl\_Duplicate","localizedErrorDetails":{"errorDetail":"Internal url 'http://google.com/' is invalid since it is already in use"},"operationResults":\[{"objectId":null,"displayName":null,"status":0,"details":"Internal url 'http://bing.com/' is invalid since it is already in use"}\],"timeStampUtc":"2017-03-23T19:50:26.465743Z","clientRequestId":"302fd775-3329-4670-a9f3-bea37004f0bb","internalTransactionId":"ea5b5475-03b9-4f08-8e95-bbb11289ab65","upn":"tperkins@f128.info","tenantId":"7918d4b5-0442-4a97-be2d-36f9f9962ece","userObjectId":"17f84be4-51f8-483a-b533-383791227a99"}```




