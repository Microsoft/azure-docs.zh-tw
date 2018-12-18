---
title: Azure CDN 即時警示 | Microsoft Docs
description: Microsoft Azure CDN 中的即時警示。 即時警示可針對 CDN 設定檔中端點的效能提供通知。
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: 1e85b809-e1a9-4473-b835-69d1b4ed3393
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: 51dce1680be5f5f4387c2ba02827195bcdbe9b48
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22998323"
---
# <a name="real-time-alerts-in-microsoft-azure-cdn"></a>Microsoft Azure CDN 中的即時警示
[!INCLUDE [cdn-premium-feature](../../includes/cdn-premium-feature.md)]

## <a name="overview"></a>概觀
本文件說明 Microsoft Azure CDN 中的即時警示。 這項功能可針對 CDN 設定檔中端點的效能提供即時通知。  您可以根據以下狀況設定電子郵件或 HTTP 警示︰

* 頻寬
* 狀態碼
* 快取狀態
* 連線

## <a name="creating-a-real-time-alert"></a>建立即時警示
1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽到您的 CDN 設定檔。
   
    ![CDN 設定檔](./media/cdn-real-time-alerts/cdn-profile-blade.png)
1. 在 [CDN 設定檔] 刀鋒視窗中，按一下 [管理]  按鈕。
   
    ![CDN 設定檔管理按鈕](./media/cdn-real-time-alerts/cdn-manage-btn.png)
   
    隨即開啟 CDN 管理入口網站。
3. 將滑鼠移至 [分析] 索引標籤上，然後將滑鼠移至 [即時統計資料] 飛出視窗上。  按一下 [即時警示] 。
   
    ![CDN 管理入口網站](./media/cdn-real-time-alerts/cdn-premium-portal.png)
   
    隨即會顯示現有警示組態 (如果有的話) 的清單。
4. 按一下 [新增警示]  按鈕。
   
    ![新增警示按鈕](./media/cdn-real-time-alerts/cdn-add-alert.png)
   
    隨即會顯示可供建立新警示的表單。
   
    ![新增警示表單](./media/cdn-real-time-alerts/cdn-new-alert.png)
5. 如果您想要在按一下 [儲存] 時讓此警示啟用，請核取 [已啟用警示] 核取方塊。
6. 在 [名稱]  欄位中輸入用來描述警示的名稱。
7. 在 [媒體類型] 下拉式清單中，選取 [HTTP 大型物件]。
   
    ![選取了 HTTP 大型物件的媒體類型](./media/cdn-real-time-alerts/cdn-http-large.png)
   
   > [!IMPORTANT]
   > 您必須選取 [HTTP 大型物件] 做為 [媒體類型]。  **來自 Verizon 的 Azure CDN** 不會使用其他選項。  若未能選取 [HTTP 大型物件]，將導致警示永不觸發。
   > 
   > 
8. 選取 [度量]、[運算子] 和 [觸發值]，來建立要監視的 [運算式]。
   
   * 針對 [度量] ，請選取您要監視的狀況類型。   是頻寬使用量，單位是每秒 Mb 數。   是連往 Edge Server 的並行 HTTP 連線數目。  如需各種快取狀態和狀態碼的定義，請參閱 [Azure CDN 快取狀態碼](https://msdn.microsoft.com/library/mt759237.aspx)和 [Azure CDN HTTP 狀態碼](https://msdn.microsoft.com/library/mt759238.aspx)
   *  是可在度量和觸發值之間建立關聯性的數學運算子。
   * **觸發值**是在傳送通知之前必須先符合的臨界值。
     
     在下列範例中，建立的運算式指出當 404 狀態碼的數目大於 25 時，就會傳送通知。
     
     ![即時警示的範例運算式](./media/cdn-real-time-alerts/cdn-expression.png)
9. 在 [間隔] 中，輸入您想要評估運算式的頻率。
10. 在 [通知時機]  下拉式清單中，選取當運算式評估為 true 時，您想要在何時收到通知。
    
    * **狀況開始**表示會在第一次偵測到指定狀況時傳送通知。
    * **結束**表示會於未再偵測到指定狀況時傳送通知。 只有在網路監視系統已偵測到發生指定狀況後，才會觸發此通知。
    * **連續**表示會在網路監視系統每一次偵測到指定狀況時傳送通知。 請記住，在每一次的間隔中，網路監視系統只會檢查一次是否發生指定狀況。
    * **開始和結束**表示會在第一次偵測到指定狀況時傳送通知，並於未再偵測到該狀況時再次傳送通知。
1. 如果您想要透過電子郵件接收通知，請核取 [透過電子郵件通知]  核取方塊。  
    
    ![透過電子郵件通知表單](./media/cdn-real-time-alerts/cdn-notify-email.png)
    
    在 [收件者]  欄位中，輸入您想要讓通知傳送到的電子郵件地址。 在 [主旨] 和 [內文] 中，您可以保留預設值，或者使用 [可用關鍵字] 清單來自訂訊息，以在傳送訊息時動態插入警示資料。
    
    > [!NOTE]
    > 您可以按一下 [測試通知] 按鈕來測試電子郵件通知，但僅限在儲存警示組態之後進行。
    > 
    > 
12. 如果您想要讓通知張貼到 Web 伺服器，請核取 [透過 HTTP Post 通知]  核取方塊。
    
    ![透過 HTTP Post 通知表單](./media/cdn-real-time-alerts/cdn-notify-http.png)
    
    在 [Url]  欄位中，輸入您想要用來張貼 HTTP 訊息的 URL。 在 [標頭]  文字方塊中，輸入要在要求中傳送的 HTTP 標頭。  在 [內文] 中，您可以使用 [可用關鍵字] 清單來自訂訊息，以在傳送訊息時動態插入警示資料。  [標頭] 和 [內文] 會預設為 XML 承載，情形類似下面的範例：
    
    ```
    <string xmlns="http://schemas.microsoft.com/2003/10/Serialization/">
        <![CDATA[Expression=Status Code : 404 per second > 25&Metric=Status Code : 404 per second&CurrentValue=[CurrentValue]&NotificationCondition=Condition Start]]>
    </string>
    ```
    
    > [!NOTE]
    > 您可以按一下 [測試通知] 按鈕來測試 HTTP Post 通知，但僅限在儲存警示組態之後進行。
    > 
    > 
13. 按一下 [儲存]  按鈕以儲存您的警示組態。  如果您在步驟 5 中核取了 [已啟用警示]  ，您的警示現在便已啟用。

## <a name="next-steps"></a>後續步驟
* 分析 [Azure CDN 中的即時統計資料](cdn-real-time-stats.md)
* 進一步了解 [進階 HTTP 報告](cdn-advanced-http-reports.md)
* 分析 [使用模式](cdn-analyze-usage-patterns.md)

